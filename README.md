import { ConnectClient, GetCurrentMetricDataCommand } from "@aws-sdk/client-connect"; // ES Modules import
import * as storeData  from "./DbConnect.mjs";
const client = new ConnectClient('eu-west-2');


export const handler = async (event, context, callback) => {
const input = { // GetCurrentMetricDataRequest
  InstanceId: "arn:aws:connect:eu-west-2:879634695871:instance/f01f9b30-5eb9-4744-8dd2-3baa9b68285c", // required
  Filters: { // Filters
    Queues: [ // Queues
      "arn:aws:connect:eu-west-2:879634695871:instance/f01f9b30-5eb9-4744-8dd2-3baa9b68285c/queue/240b0ef7-708f-40fa-bf87-1c1f7b458906",
      "arn:aws:connect:eu-west-2:879634695871:instance/f01f9b30-5eb9-4744-8dd2-3baa9b68285c/queue/2fcc1817-6fb6-477a-be1e-88b5c1d3c98a"
    ],
    Channels: [ // Channels
      "VOICE",
    ],
  },
  Groupings: [ // Groupings
    "QUEUE",
  ],
  CurrentMetrics: [ // CurrentMetrics // required
    { // CurrentMetric
      Name: "AGENTS_ONLINE",
      Unit: "COUNT",
    },
    { // CurrentMetric
      Name: "AGENTS_ON_CALL",
      Unit: "COUNT",
    },
    { // CurrentMetric
      Name: "CONTACTS_IN_QUEUE",
      Unit: "COUNT",
    },
    { // CurrentMetric
      Name: "AGENTS_STAFFED",
      Unit: "COUNT",
    }
  ],
  MaxResults: Number("2"),
};
    let records = {};
    const command = new GetCurrentMetricDataCommand(input);
    const response = await client.send(command).then(async(responseData) => {
      console.log('responseData : ', responseData);
        await storeData.runFunction(responseData).then(obj => {
         records['data'] = obj ;
         callback(null, records);//return records;
        }).catch(err=> {
          console.log('inside promsie err', err);
          records['data'] = err ;
          callback(err, "data: Error Occured");
        });
    }).catch(err=> {
        console.log('inside promsie err of responsedata', err);
        records['data'] = err ;
        callback(err, "data: Error Occured");
    });
};


this is index page below is dbckonnect.mjs

import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import {
  DynamoDBDocumentClient,
  PutCommand
} from "@aws-sdk/lib-dynamodb";

const client = new DynamoDBClient({});
const dynamo = DynamoDBDocumentClient.from(client);

async function runFunction(context){
    let finalData = {};
      if (context.MetricResults.length > 0) {
        for(let [index,wallBoardIndex] of context.MetricResults.entries()) {
                    let wallBoardData = {};
                    wallBoardIndex.Collections.forEach((data)=> {
                        // console.log(`Metric data for record `, data);
                        wallBoardData[data.Metric.Name] = data.Value;
                    });
                    const tableName = "wb_wallboard_data";
                    try {
                      await dynamo.send(
                      new PutCommand({
                        TableName: tableName,
                        Item: {
                          "QueueArn": wallBoardIndex.Dimensions.Queue.Arn,
                          "AGENTS_ONLINE": wallBoardData.AGENTS_ONLINE,
                          "AGENTS_ON_CALL": wallBoardData.AGENTS_ON_CALL,
                          "CONTACTS_IN_QUEUE": wallBoardData.CONTACTS_IN_QUEUE,
                          "AGENTS_STAFFED": wallBoardData.AGENTS_STAFFED
                          },
                        })
                      );
                      if(index == (context.MetricResults.length - 1)) {
                        return new Promise((resolve, reject) => {resolve(finalData['data'] = 'Data inserted')});
                      }
                    }
                    catch(err) {
                      return new Promise((resolve, reject) => {reject(finalData['data'] = err)});
                    };
        
        }
      } else {
        return new Promise((resolve, reject) => {resolve(finalData['data'] = 'NO RECORDS FETCH')});
      }
};



export {
  runFunction
};


can u modify the below function taking reference to the above one 


import { ConnectClient, GetMetricDataCommand } from "@aws-sdk/client-connect";
import * as storeData from "./DbConnect.mjs";
export const handler = async (event, context, callback) => {
  const queue1 = 'arn:aws:connect:eu-west-2:879634695871:instance/f01f9b30-5eb9-4744-8dd2-3baa9b68285c/queue/2fcc1817-6fb6-477a-be1e-88b5c1d3c98a';
  const queue2 = 'arn:aws:connect:eu-west-2:879634695871:instance/f01f9b30-5eb9-4744-8dd2-3baa9b68285c/queue/240b0ef7-708f-40fa-bf87-1c1f7b458906';
  const client = new ConnectClient('eu-west-2');
  // Rounding the current time to the nearest multiple of 5
  const currentTime = new Date();
  const roundedMinutes = Math.round(currentTime.getMinutes() / 5) * 5;
  const startTime = new Date(currentTime.getFullYear(), currentTime.getMonth(), currentTime.getDate(), currentTime.getHours(), roundedMinutes);

  //  end time is after start time and is a multiple of 5 minutes
  const endTime = new Date(startTime.getTime() + (5 * 60 * 1000)); // Add 5 minutes

 
 
  const input = {
    InstanceId: "arn:aws:connect:eu-west-2:879634695871:instance/f01f9b30-5eb9-4744-8dd2-3baa9b68285c",
    StartTime: startTime,
    EndTime: endTime,
    Filters: {
      Queues: [queue1, queue2]
    },
    HistoricalMetrics: [
      {
        Name: "CONTACTS_HANDLED",
        Statistic: "SUM",
        Unit: "COUNT"
      },
      {
        Name: "QUEUED_TIME",
        Statistic: "MAX",
        Unit: "SECONDS"
      }
    ]
  };
  const command = new GetMetricDataCommand(input);
  const response = await client.send(command);
  console.log(response.MetricResults)
  let putData1 = null;
  let putData2 = null;
  if (response && response.MetricResults && response.MetricResults.length > 0) {
    const result1 = response.MetricResults[0].Collections.reduce((acc, item) => {
      acc[item.Metric.Name] = item.Value;
      return acc;
    }, {});
    const context1 = {
      CONTACTS_HANDLED: result1.CONTACTS_HANDLED,
      MAX_QUEUED_TIME: result1.MAX_QUEUED_TIME
    };
    putData1 = await storeData.runFunction(context1, queue1);
    if (response.MetricResults.length > 1) {
      const result2 = response.MetricResults[1].Collections.reduce((acc, item) => {
        acc[item.Metric.Name] = item.Value;
        return acc;
      }, {});
      const context2 = {
        CONTACTS_HANDLED: result2.CONTACTS_HANDLED,
        MAX_QUEUED_TIME: result2.MAX_QUEUED_TIME
      };
      putData2 = await storeData.runFunction(context2, queue2);
    }
  }
  console.log("putData1:", putData1);
  console.log("putData2:", putData2);
  return { putData1, putData2 };
};



function roundToNearest5Minutes(date) {
  const minutes = date.getMinutes();
  const roundedMinutes = Math.ceil(minutes / 5) * 5;
  date.setMinutes(roundedMinutes);
  return date;
}

import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import {
  DynamoDBDocumentClient,
  PutCommand
} from "@aws-sdk/lib-dynamodb";

const client = new DynamoDBClient({});
const dynamo = DynamoDBDocumentClient.from(client);

async function runFunction(context,queue){
    let finalData = {};
      console.log('inside ()', context,queue,context.CONTACTS_HANDLED);

 const tableName = "wb_wallboard_data";
 try {
                     const res =  await dynamo.send(
                      new PutCommand({
                        TableName: tableName,
                         Item: {
         "QueueArn": queue,
         "CONTACTS_HANDLED": context.CONTACTS_HANDLED || 0, // Provide default value if property is missing
         "MAX_QUEUED_TIME": context.MAX_QUEUED_TIME || 0, // Provide default value if property is missing
       }
                        })
                      );
                      console.log(res)
                      
                      return res;
                    }
                    catch(err) {
                    //   return new Promise((resolve, reject) => {reject(finalData['data'] = err)});
                    
                    console.error("error in q2:", err);
                    
                    return null;
                    };

}
  


export {
  runFunction
};
