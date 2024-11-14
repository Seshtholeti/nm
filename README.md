import { ConnectClient, GetCurrentMetricDataCommand, ListQueuesCommand, ListUsersCommand } from "@aws-sdk/client-connect";
// Initialize the Connect client
const client = new ConnectClient({ region: 'us-east-1' });
// Helper function to get an array of dates between two dates
function getDateRange(startDate, endDate) {
 const dates = [];
 let currentDate = new Date(startDate);
 while (currentDate <= new Date(endDate)) {
   dates.push(new Date(currentDate).toISOString().split("T")[0]); // Get only the date part (YYYY-MM-DD)
   currentDate.setDate(currentDate.getDate() + 1); // Move to the next day
 }
 return dates;
}
// Helper function to fetch all queues dynamically
async function getQueues(instanceId) {
 const input = { InstanceId: instanceId };
 try {
   const command = new ListQueuesCommand(input);
   const data = await client.send(command);
   const queueIds = data.QueueSummaryList.map((queue) => queue.Id);
   return queueIds;
 } catch (err) {
   console.error("Error fetching queues:", err);
   throw err;
 }
}
// Helper function to fetch all agents dynamically
async function getAgents(instanceId) {
 const input = { InstanceId: instanceId };
 try {
   const command = new ListUsersCommand(input);
   const data = await client.send(command);
   const agentIds = data.UserSummaryList.map((user) => user.Id);
   return agentIds;
 } catch (err) {
   console.error("Error fetching agents:", err);
   throw err;
 }
}
export const handler = async (event) => {
 const { resourceType, selectedResources, startDate, endDate } = event;
 // Validate that selectedResources is not empty
 if (!selectedResources || selectedResources.length === 0) {
   throw new Error("You must select at least one resource.");
 }
 const instanceId = process.env.InstanceId;
 const dateRange = getDateRange(startDate, endDate);
 const response = {
   resourceType,
   startDate,
   endDate,
   selectedResources,
   metricsByDate: [],
 };
 // Fetch metrics for both Queues and Agents if specified
 for (const date of dateRange) {
   const dateMetrics = { date, queueMetrics: [], agentMetrics: [] };
   // Fetching Queue Metrics
   if (resourceType === "Queues") {
     const queueInput = {
       InstanceId: instanceId,
       Filters: {
         Queues: selectedResources,
         Channels: ["VOICE"],
       },
       Groupings: ["QUEUE"],
       CurrentMetrics: [
         { Name: "AGENTS_ONLINE", Unit: "COUNT" },
         { Name: "AGENTS_AVAILABLE", Unit: "COUNT" },
         { Name: "CONTACTS_IN_QUEUE", Unit: "COUNT" },
         { Name: "AGENTS_ERROR", Unit: "COUNT" },
         { Name: "CONTACTS_SCHEDULED", Unit: "COUNT" },
         { Name: "OLDEST_CONTACT_AGE", Unit: "SECONDS" },
       ],
       StartTime: new Date(`${date}T00:00:00Z`).toISOString(),
       EndTime: new Date(`${date}T23:59:59Z`).toISOString(),
     };
     try {
       const queueCommand = new GetCurrentMetricDataCommand(queueInput);
       const queueData = await client.send(queueCommand);
       if (queueData.MetricResults && queueData.MetricResults.length > 0) {
         queueData.MetricResults.forEach((result) => {
           const metrics = result.Collections.map((collection) => ({
             metricName: collection.Metric.Name,
             metricValue: collection.Value,
           }));
           dateMetrics.queueMetrics.push({
             queueId: result.Dimensions.Queue.Id,
             queueName: result.Dimensions.Queue.Name || "Unknown Queue",
             metrics,
           });
         });
       }
     } catch (err) {
       console.error(`Error fetching queue metrics for date ${date}:`, err);
       dateMetrics.queueMetrics.push({
         error: err.message,
       });
     }
   }
   // Fetching Agent Metrics
   if (resourceType === "Agents") {
     const agentInput = {
       InstanceId: instanceId,
       Filters: {
         RoutingProfileIds: selectedResources,
         Channels: ["VOICE"],
       },
       Groupings: ["AGENT","QUEUE"],
       CurrentMetrics: [
         { Name: "AGENTS_AFTER_CONTACT_WORK", Unit: "COUNT" },
         { Name: "AGENTS_AVAILABLE", Unit: "COUNT" },
         { Name: "AGENTS_ON_CALL", Unit: "COUNT" },
         { Name: "AGENTS_STAFFED", Unit: "COUNT" },
         { Name: "CONTACTS_HANDLED", Unit: "COUNT" },
         { Name: "OCCUPANCY", Unit: "PERCENT" },
       ],
       StartTime: new Date(`${date}T00:00:00Z`).toISOString(),
       EndTime: new Date(`${date}T23:59:59Z`).toISOString(),
     };
     try {
       const agentCommand = new GetCurrentMetricDataCommand(agentInput);
       const agentData = await client.send(agentCommand);
       if (agentData.MetricResults && agentData.MetricResults.length > 0) {
         agentData.MetricResults.forEach((result) => {
           const metrics = result.Collections.map((collection) => ({
             metricName: collection.Metric.Name,
             metricValue: collection.Value,
           }));
           dateMetrics.agentMetrics.push({
             agentId: result.Dimensions.Agent.Id,
             agentName: result.Dimensions.Agent.Name || "Unknown Agent",
             metrics,
           });
         });
       }
     } catch (err) {
       console.error(`Error fetching agent metrics for date ${date}:`, err);
       dateMetrics.agentMetrics.push({
         error: err.message,
       });
     }
   }
   response.metricsByDate.push(dateMetrics);
 }
 return response;
};

this is the error in console
{
  "resourceType": "Agents",
  "startDate": "2024-11-6",
  "endDate": "2024-11-11",
  "selectedResources": [
    "f8c742b9-b5ef-4948-8bbf-9a33c892023f"
  ],
  "metricsByDate": [
    {
      "date": "2024-11-06",
      "queueMetrics": [],
      "agentMetrics": [
        {
          "error": "Invalid request body"
        }
      ]
    },
    {
      "date": "2024-11-07",
      "queueMetrics": [],
      "agentMetrics": [
        {
          "error": "Invalid request body"
        }
      ]
    },
    {
      "date": "2024-11-08",
      "queueMetrics": [],
      "agentMetrics": [
        {
          "error": "Invalid request body"
        }
      ]
    },
    {
      "date": "2024-11-09",
      "queueMetrics": [],
      "agentMetrics": [
        {
          "error": "Invalid request body"
        }
      ]
    },
    {
      "date": "2024-11-10",
      "queueMetrics": [],
      "agentMetrics": [
        {
          "error": "Invalid request body"
        }
      ]
    },
    {
      "date": "2024-11-11",
      "queueMetrics": [],
      "agentMetrics": [
        {
          "error": "Invalid request body"
        }
      ]
    }
  ]
}
