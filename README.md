// import { ConnectClient, ListUsersCommand, GetCurrentMetricDataCommand } from "@aws-sdk/client-connect";
// //  the Connect client
// const client = new ConnectClient({ region: 'us-east-1' });
// //  function to get an array of dates between two dates
// function getDateRange(startDate, endDate) {
//  const dates = [];
//  let currentDate = new Date(startDate);
//  while (currentDate <= new Date(endDate)) {
//    dates.push(new Date(currentDate).toISOString().split("T")[0]);
//    currentDate.setDate(currentDate.getDate() + 1);
//  }
//  return dates;
// }
// //  function to aggregate metrics
// function aggregateMetrics(dailyMetrics) {
//  const aggregated = {};
//  dailyMetrics.forEach((day) => {
//    day.metrics.forEach((metric) => {
//      metric.metrics.forEach(({ metricName, metricValue }) => {
//        if (!aggregated[metricName]) {
//          aggregated[metricName] = 0;
//        }
//        aggregated[metricName] += metricValue; // Sum up the values
//      });
//    });
//  });
//  // Convert aggregated metrics to array format
//  return Object.entries(aggregated).map(([metricName, metricValue]) => ({
//    metricName,
//    metricValue,
//  }));
// }
// // Main handler function
// export const handler = async (event) => {
//  const { resourceType, selectedResources, startDate, endDate, isCumulativeReport } = event;
//  // Validate that selectedResources is not empty
//  if (!selectedResources || selectedResources.length === 0) {
//    return { error: "You must select at least one resource." };
//  }
//  const instanceId = process.env.InstanceId;
//  const response = {
//    resourceType,
//    startDate,
//    endDate,
//    selectedResources,
//    metricsByDate: [],
//    cumulativeMetrics: {},
//  };
//  const dateRange = getDateRange(startDate, endDate);
//  // Function to fetch metrics
//  const fetchMetrics = async (filters, groupings, metrics, type) => {
//    const dailyMetrics = [];
//    for (const date of dateRange) {
//      const input = {
//        InstanceId: instanceId,
//        Filters: filters,
//        Groupings: groupings,
//        CurrentMetrics: metrics,
//        StartTime: new Date(`${date}T00:00:00Z`).toISOString(),
//        EndTime: new Date(`${date}T23:59:59Z`).toISOString(),
//      };
//      try {
//        const command = new GetCurrentMetricDataCommand(input);
//        const data = await client.send(command);
//        const metricsForDate = {
//          date,
//          type,
//          metrics: [],
//        };
//        if (data.MetricResults && data.MetricResults.length > 0) {
//          data.MetricResults.forEach((result) => {
//            const resultMetrics = result.Collections.map((collection) => ({
//              metricName: collection.Metric.Name,
//              metricValue: collection.Value,
//            }));
//            metricsForDate.metrics.push({
//              queueId: result.Dimensions.Queue.Id || "Unknown Queue",
//              queueName: result.Dimensions.Queue.Name || "Unknown Queue",
//              metrics: resultMetrics,
//            });
//          });
//        }
//        dailyMetrics.push(metricsForDate);
//      } catch (err) {
//        console.error(`Error fetching metrics for date ${date}:`, err);
//        dailyMetrics.push({
//          date,
//          errorMessage: err.message,
//        });
//      }
//    }
//    return dailyMetrics;
//  };
//  // Check for Cumulative Report flag first
//  if (isCumulativeReport) {
//    // Cumulative report for Agents
//    if (resourceType === "Agents") {
//      try {
//        const listUsersCommand = new ListUsersCommand({ InstanceId: instanceId });
//        const usersResponse = await client.send(listUsersCommand);
//        const agents = usersResponse.UserSummaryList;
//        const filteredAgents = agents.filter((agent) => selectedResources.includes(agent.UserId));
//        const filters = {
//          Agents: filteredAgents.map((agent) => agent.Arn),
//          Queues: selectedResources,
//          Channels: ["VOICE"],
//        };
//        const groupings = ["QUEUE", "AGENTS"];
//        const metrics = [
//          { Name: "AGENTS_ONLINE", Unit: "COUNT" },
//          { Name: "AGENTS_AVAILABLE", Unit: "COUNT" },
//          { Name: "CONTACTS_HANDLED", Unit: "COUNT" },
//        ];
//        const dailyMetrics = await fetchMetrics(filters, groupings, metrics, "Agent");
//        response.cumulativeMetrics = aggregateMetrics(dailyMetrics);
//        return response; // Return only cumulative report
//      } catch (error) {
//        return { statusCode: 500, body: JSON.stringify({ error: error.message }) };
//      }
//    }
//    // Cumulative report for Queues
//    else if (resourceType === "Queues") {
//      const filters = {
//        Queues: selectedResources,
//        Channels: ["VOICE"],
//      };
//      const groupings = ["QUEUE"];
//      const metrics = [
//        { Name: "AGENTS_ONLINE", Unit: "COUNT" },
//        { Name: "AGENTS_AVAILABLE", Unit: "COUNT" },
//        { Name: "CONTACTS_IN_QUEUE", Unit: "COUNT" },
//        { Name: "AGENTS_ERROR", Unit: "COUNT" },
//        { Name: "CONTACTS_SCHEDULED", Unit: "COUNT" },
//        { Name: "OLDEST_CONTACT_AGE", Unit: "SECONDS" },
//      ];
//      const dailyMetrics = await fetchMetrics(filters, groupings, metrics, "Queue");
//      response.cumulativeMetrics = aggregateMetrics(dailyMetrics);
//      return response; // Return only cumulative report
//    }
//  } else {
//    // Daily report for Agents
//    if (resourceType === "Agents") {
//      try {
//        const listUsersCommand = new ListUsersCommand({ InstanceId: instanceId });
//        const usersResponse = await client.send(listUsersCommand);
//        const agents = usersResponse.UserSummaryList;
//        const filteredAgents = agents.filter((agent) => selectedResources.includes(agent.UserId));
//        const filters = {
//          Agents: filteredAgents.map((agent) => agent.Arn),
//          Queues: selectedResources,
//          Channels: ["VOICE"],
//        };
//        const groupings = ["QUEUE", "AGENTS"];
//        const metrics = [
//          { Name: "AGENTS_ONLINE", Unit: "COUNT" },
//          { Name: "AGENTS_AVAILABLE", Unit: "COUNT" },
//          { Name: "CONTACTS_HANDLED", Unit: "COUNT" },
//        ];
//        response.metricsByDate = await fetchMetrics(filters, groupings, metrics, "Agent");
//        return response;
//      } catch (error) {
//        return { statusCode: 500, body: JSON.stringify({ error: error.message }) };
//      }
//    }
//    // Daily report for Queues
//    else if (resourceType === "Queues") {
//      const filters = {
//        Queues: selectedResources,
//        Channels: ["VOICE"],
//      };
//      const groupings = ["QUEUE"];
//      const metrics = [
//        { Name: "AGENTS_ONLINE", Unit: "COUNT" },
//        { Name: "AGENTS_AVAILABLE", Unit: "COUNT" },
//        { Name: "CONTACTS_IN_QUEUE", Unit: "COUNT" },
//        { Name: "AGENTS_ERROR", Unit: "COUNT" },
//        { Name: "CONTACTS_SCHEDULED", Unit: "COUNT" },
//        { Name: "OLDEST_CONTACT_AGE", Unit: "SECONDS" },
//      ];
//      response.metricsByDate = await fetchMetrics(filters, groupings, metrics, "Queue");
//      return response;
//    }
//  }
//  return { error: "Unsupported resource type" };
// };
