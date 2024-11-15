import { ConnectClient, ListUsersCommand, GetCurrentMetricDataCommand } from "@aws-sdk/client-connect";
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";
import { v4 as uuidv4 } from 'uuid';
import { S3RequestPresigner } from "@aws-sdk/s3-request-presigner";
import { parse } from 'json2csv';
import { Readable } from 'stream';
// Initialize clients
const connectClient = new ConnectClient({ region: 'us-east-1' });
const s3Client = new S3Client({ region: 'us-east-1' });
// Helper function to get an array of dates between two dates
function getDateRange(startDate, endDate) {
 const dates = [];
 let currentDate = new Date(startDate);
 while (currentDate <= new Date(endDate)) {
   dates.push(new Date(currentDate).toISOString().split("T")[0]);
   currentDate.setDate(currentDate.getDate() + 1);
 }
 return dates;
}
// Helper function to aggregate metrics
function aggregateMetrics(dailyMetrics) {
 const aggregated = {};
 dailyMetrics.forEach((day) => {
   day.metrics.forEach((metric) => {
     metric.metrics.forEach(({ metricName, metricValue }) => {
       if (!aggregated[metricName]) {
         aggregated[metricName] = 0;
       }
       aggregated[metricName] += metricValue;
     });
   });
 });
 return Object.entries(aggregated).map(([metricName, metricValue]) => ({ metricName, metricValue }));
}
// Function to upload file to S3 and return presigned URL
async function uploadFileToS3(data, fileName, fileFormat) {
 const bucketName = process.env.BUCKET_NAME;
 const fileKey = `${uuidv4()}.${fileFormat}`;
 // Upload file
 const putCommand = new PutObjectCommand({
   Bucket: bucketName,
   Key: fileKey,
   Body: data,
   ContentType: fileFormat === 'json' ? 'application/json' : 'text/csv'
 });
 await s3Client.send(putCommand);
 // Generate presigned URL
 const presigner = new S3RequestPresigner({ s3Client });
 const presignedUrl = await presigner.presignGetObject({ Bucket: bucketName, Key: fileKey });
 return presignedUrl;
}
// Main handler function
export const handler = async (event) => {
 const { resourceType, selectedResources, startDate, endDate, isCumulativeReport, fileFormat } = event;
 if (!selectedResources || selectedResources.length === 0) {
   return { error: "You must select at least one resource." };
 }
 const instanceId = process.env.InstanceId;
 const response = {
   resourceType,
   startDate,
   endDate,
   selectedResources,
   metricsByDate: [],
   cumulativeMetrics: {}
 };
 const dateRange = getDateRange(startDate, endDate);
 // Check for cumulative report flag first
 const isCumulative = isCumulativeReport;
 const isCSV = fileFormat === 'csv';
 const fetchMetrics = async (filters, groupings, metrics, type) => {
   const dailyMetrics = [];
   for (const date of dateRange) {
     const input = {
       InstanceId: instanceId,
       Filters: filters,
       Groupings: groupings,
       CurrentMetrics: metrics,
       StartTime: new Date(`${date}T00:00:00Z`).toISOString(),
       EndTime: new Date(`${date}T23:59:59Z`).toISOString()
     };
     try {
       const command = new GetCurrentMetricDataCommand(input);
       const data = await connectClient.send(command);
       const metricsForDate = {
         date,
         type,
         metrics: []
       };
       if (data.MetricResults && data.MetricResults.length > 0) {
         data.MetricResults.forEach((result) => {
           const resultMetrics = result.Collections.map((collection) => ({
             metricName: collection.Metric.Name,
             metricValue: collection.Value
           }));
           metricsForDate.metrics.push({
             queueId: result.Dimensions.Queue.Id || "Unknown Queue",
             queueName: result.Dimensions.Queue.Name || "Unknown Queue",
             metrics: resultMetrics
           });
         });
       }
       dailyMetrics.push(metricsForDate);
     } catch (err) {
       console.error(`Error fetching metrics for date ${date}:`, err);
       dailyMetrics.push({
         date,
         errorMessage: err.message
       });
     }
   }
   return dailyMetrics;
 };
 const filters = {
   Queues: selectedResources,
   Channels: ["VOICE"]
 };
 const groupings = ["QUEUE"];
 const metrics = [
   { Name: "AGENTS_ONLINE", Unit: "COUNT" },
   { Name: "AGENTS_AVAILABLE", Unit: "COUNT" },
   { Name: "CONTACTS_IN_QUEUE", Unit: "COUNT" }
 ];
 // Fetch and aggregate metrics
 const dailyMetrics = await fetchMetrics(filters, groupings, metrics, resourceType);
 if (isCumulative) {
   response.cumulativeMetrics = aggregateMetrics(dailyMetrics);
 } else {
   response.metricsByDate = dailyMetrics;
 }
 // Generate file data
 const fileData = isCSV
   ? parse(isCumulative ? response.cumulativeMetrics : response.metricsByDate)
   : JSON.stringify(isCumulative ? response.cumulativeMetrics : response.metricsByDate);
 const fileName = isCumulative ? 'cumulative' : 'daily';
 const downloadUrl = await uploadFileToS3(fileData, fileName, fileFormat);
 return {
   downloadUrl,
   message: "Download report using this URL"
 };
};
