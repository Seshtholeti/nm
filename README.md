// import { ConnectClient, GetCurrentMetricDataCommand } from "@aws-sdk/client-connect";
// // Initialize the Connect client
// const client = new ConnectClient({ region: 'us-east-1' });
// export const handler = async (event) => {
//  const { resourceType, selectedResources, startDate, endDate } = event;
//  // Validate that selectedResources is not empty
//  if (!selectedResources || selectedResources.length === 0) {
//    return { error: "You must select at least one resource." };
//  }
//  const instanceId = process.env.InstanceId;  // Use InstanceId from environment variables
//  const response = {
//    resourceType,
//    startDate,
//    endDate,
//    selectedResources,
//    metrics: []
//  };
//  if (resourceType === "Queues") {
//    const queueIds = selectedResources; // Queue IDs passed from the front-end
//    const input = {
//      InstanceId: instanceId,
//      Filters: {
//        Queues: queueIds,  // Pass selected queue IDs
//        Channels: ["VOICE"],  // Adjust channels if necessary
//      },
//      Groupings: ["QUEUE"],  // Grouping by queue
//      CurrentMetrics: [
//        { Name: "AGENTS_AFTER_CONTACT_WORK", Unit: "COUNT" },
//                { Name: "AGENTS_ON_CALL", Unit: "COUNT" },
//                { Name: "AGENTS_AVAILABLE", Unit: "COUNT" },
//                { Name: "AGENTS_ONLINE", Unit: "COUNT" },
//                { Name: "AGENTS_STAFFED", Unit: "COUNT" },
//                { Name: "CONTACTS_IN_QUEUE", Unit: "COUNT" },
//                { Name: "AGENTS_ERROR", Unit: "COUNT" },
//                { Name: "AGENTS_NON_PRODUCTIVE", Unit: "COUNT" },
//                { Name: "AGENTS_ON_CONTACT", Unit: "COUNT" },
//                { Name: "CONTACTS_SCHEDULED", Unit: "COUNT" },
//                { Name: "OLDEST_CONTACT_AGE", Unit: "SECONDS" },
//                { Name: "SLOTS_ACTIVE", Unit: "COUNT" },
//                { Name: "SLOTS_AVAILABLE", Unit: "COUNT" },
//        // Add other metrics as needed
//      ],
//      StartTime: new Date(startDate).toISOString(),
//      EndTime: new Date(endDate).toISOString(),
//    };
//    try {
//      const command = new GetCurrentMetricDataCommand(input);
//      const data = await client.send(command);
//      if (data.MetricResults && data.MetricResults.length > 0) {
//        data.MetricResults.forEach((result) => {
//          const queueMetrics = result.Collections.map((collection) => ({
//            metricName: collection.Metric.Name,
//            metricValue: collection.Value,
//          }));
//          response.metrics.push({
//            queueId: result.Dimensions.Queue.Id,
//            queueName: result.Dimensions.Queue.Name || "Unknown Queue",
//            metrics: queueMetrics,
//          });
//        });
//      }
//      // Return the required structure
//      return response;
//    } catch (err) {
//      console.error("Error fetching metrics:", err);
//      return { error: err.message };
//    }
//  } else {
//    return { error: "Unsupported resource type" };
//  }
// };
