---
title: LOAD TESTING RESEARCH
categories: [ C# , .NET , JavaScript  , devops.]
tags: [Web Development,Docker,load testing,Automation , C# , .NET ,JavaScript  ,  dotnet #angular
] # TAG names should always be lowercase
image: https://i.imgur.com/Mh0q0nX.jpg
---
<!-- ![Alt Text](https://i.imgur.com/Mh0q0nX.jpg){:width="900px"height="200px"} -->
# Load Balancing And Testing Research

Load testing is an essential part of the software development process to identify performance bottlenecks, improve scalability, and ensure that applications can handle the expected traffic. Traditional load testing tools can be difficult to use, require a significant amount of setup time, and can be expensive to license. This can create a barrier for developers and DevOps teams to incorporate load testing into their workflow, which can lead to performance issues going unnoticed until they are encountered in production.

To address this problem, there are several load testing tools available, such as K6, NBomber, JMeter, and Gatling. These tools provide developers and DevOps teams with a simple and effective way to test the performance and scalability of their applications.

K6 is one of the open-source, developer-centric load testing tools available. It is designed to help developers and DevOps teams test the performance and scalability of their applications. Built on top of the Go programming language, K6 provides a powerful scripting language for creating test scenarios.

### Load Testing Tools:
There are various load testing tools available in the market, including K6, NBomber, JMeter, Gatling, and Locust, among others. These tools can be used to generate and simulate a large number of virtual users to test the performance and scalability of applications. In this documentation, we will focus on using K6 to test the UNIT TRUST API.

### The Problem Load Testing tools Solves
Applications can become slow and unresponsive under heavy loads, which can result in a poor user experience and lost revenue for the business. Load testing is an essential part of the software development process to identify performance bottlenecks, improve scalability, and ensure that applications can handle the expected traffic.
### Goal of the Load Testing
The ultimate goal of the load testing implemented in the provided code is to evaluate and ensure the reliability, scalability, and overall performance of the investment platform's API under simulated user interactions and various scenarios. Load testing is employed to simulate realistic usage patterns, gauge the system's response under different levels of load, and identify any potential bottlenecks or performance degradation.

Specifically, the load testing aims to achieve the following objectives:

#### Performance Evaluation:
 The code simulates a real-world scenario where multiple users interact with the investment platform simultaneously. By doing so, it assesses the platform's response times and overall performance under different levels of load.

#### Scalability Assessment:
 The test scenarios are designed to evaluate how well the platform can scale to accommodate a growing number of concurrent users. This is important to ensure that the system can handle increased user demand without compromising its performance.

#### Stress Testing:
 The code applies stress to the system by generating a significant number of requests and interactions. This helps identify the breaking points and limitations of the system, allowing for proactive measures to be taken to prevent crashes or failures.

#### Concurrency Analysis:
 The code assesses the system's ability to handle a high number of concurrent connections. This is crucial to ensure that the platform's resources, such as database connections, are efficiently managed without causing contention.

#### Error Detection:
 By checking HTTP response codes and patterns, the code identifies potential issues such as incorrect request handling, authentication failures, or server errors. Detecting and addressing these issues early is essential for maintaining the platform's reliability.

#### Data Consistency:
 Through various interactions like purchases, balance checks, and sales, the code assesses the accuracy and consistency of data returned by the API. Any discrepancies could indicate underlying issues that need to be addressed.

#### Reporting and Analysis:
 The code generates comprehensive reports, including CSV, HTML, and text summaries. These reports provide insights into the system's performance, potential bottlenecks, and areas for improvement. They also serve as documentation for the load testing process.

By achieving these objectives, the load testing enables the identification of potential performance bottlenecks, scalability limitations, and other issues that may impact the investment platform's users. Addressing these issues based on load testing results can enhance the platform's stability, user experience, and overall reliability.

#### What we are trying to achieve:
We require a solution to thoroughly assess the performance of the API endpoints that are responsible for tasks such as investor registration, joining investors into a unit trust fund, purchasing units, and selling units. Our aim is to understand how these endpoints perform under heavy usage. By simulating a substantial amount of traffic using virtual users, we can gain insights into how these endpoints handle high loads and pinpoint any potential performance-related issues.

To achieve this, we can leverage the capabilities of K6 and other load testing tools. The provided script exemplifies the application of K6 in addressing this challenge. The script showcases how to utilize K6 to add new investors to the UNIT TRUST API, subsequently enrolling them in a fund. The script employs the Faker library to generate randomized data and employs K6's built-in HTTP library to send requests to the API endpoints. For diagnostic purposes, the script logs both the response status and body.

It's worth noting that the approach isn't exclusive to K6. Other load testing tools like NBomber can also be utilized to attain comparable outcomes. This script serves as a clear demonstration of how we can employ K6, or similar tools, to effectively analyze the performance of our API endpoints under demanding conditions.

Here is the K6 script:  
```javascript
import http from "k6/http";
import { sleep } from "k6";
import { htmlReport } from "https://raw.githubusercontent.com/benc-uk/k6-reporter/main/dist/bundle.js";
import faker from "https://cdnjs.cloudflare.com/ajax/libs/Faker/3.1.0/faker.min.js";
import papaparse from 'https://cdn.jsdelivr.net/npm/papaparse@5.3.0';
import dayjs from "https://cdn.skypack.dev/dayjs@1.10.7";
import { check } from 'k6';
import config from './config.js';
import { textSummary } from "https://jslib.k6.io/k6-summary/0.0.1/index.js";


let accessToken = "";  // Store the access token
let tokenExpiration = 0;  // Store the token expiration time

// Function to get the access token
function getAccessToken() {
  const tokenPayload = {
    grant_type: "client_credentials",
    client_id: config.token.clientID,
    client_secret: config.token.clientSecret,
    scope: config.token.scope,
  };

  const tokenResponse = http.post(config.endpoints.authServerUrl + "/connect/token", tokenPayload);
  const tokenData = JSON.parse(tokenResponse.body);

  accessToken = tokenData.access_token;
  tokenExpiration = Math.floor(Date.now() / 1000) + tokenData.expires_in;  // Store the expiration time in seconds
}
// Function to check and refresh the token if needed
function ensureAccessToken() {
  if (accessToken === "" || tokenExpiration - Math.floor(Date.now() / 1000) < 300) {
    getAccessToken();
  } else if (tokenExpiration - Math.floor(Date.now() / 1000) < 360) {  // Refresh 5 minutes before expiration
    getAccessToken();
  }
}

const investors = [];
let hourCounter = 0;

export let options = {
  vus: config.vus,
  duration: config.duration,
};

export default function () {
  ensureAccessToken();  // Check and refresh token if needed



  const investorPayload = {
    IDNumber: faker.random.number({ min: 10000000, max: 99999999 }),
    FirstName: faker.name.firstName(),
    Surname: faker.name.lastName(),
    DateOfBirth: faker.date.past(30, new Date('2000-01-01')).toLocaleDateString('en-US', { day: '2-digit', month: 'short', year: 'numeric' }),
    PhoneNumber: faker.phone.phoneNumber('+254#########'),
    EmailAddress: faker.internet.email(),
    PIN: `${faker.random.alphaNumeric(2)}${faker.random.number({ min: 100000, max: 999999 })}${faker.random.alphaNumeric(2)}`,
    ClientIdentifier: `${faker.phone.phoneNumber('+254#########')}-${faker.random.number({ min: 10000000, max: 99999999 })}`,
    FundsSourceCode: null,
    AgentID: null,
    BankAccount: {
      Bank: 'KENYA COMMERCIAL BANK LTD',
      BankBranch: '0000001179 ',
      AccountName: faker.name.findName(),
      AccountNumber: faker.finance.account(),
      Currency: 'KES'
    }
  };

  const fundPayload = {
    ClientIdentifier: investorPayload.ClientIdentifier,
    FundShareClassCode: "MMF",
    IncomeDistributionMode: faker.random.number({ min: 1, max: 3 }).toString()
  };

  const headers = {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${accessToken}`,
  };

  const investorUrl = `${config.endpoints.investorUrl}?api-version=${config.versions.investor}`;
  const fundUrl = `${config.endpoints.fundUrl}?api-version=${config.versions.fund}`;
  const purchaseUrl = `${config.endpoints.purchaseUrl}?api-version=${config.versions.purchase}`;
  const balanceUrl = `${config.endpoints.balanceUrl}?api-version=${config.versions.balance}`;
  const saleUrl = `${config.endpoints.saleUrl}?api-version=${config.versions.sale}`;

  // Register investor
  const investorRes = http.post(investorUrl, JSON.stringify(investorPayload), { headers: headers });
  check(investorRes, {
    'Investor creation successful': (res) => res.status === 201,
    'Bad Request (400)': (res) => res.status === 400,
    'Unauthorized (401)': (res) => res.status === 401,
    'Internal Server Error (500)': (res) => res.status === 500,
    'Request Timeout (408)': (res) => res.status === 408,
  });
  console.log(investorRes.status);
  console.log(investorRes.body);

  // Join fund
  const fundRes = http.post(fundUrl, JSON.stringify(fundPayload), { headers: headers });
  check(fundRes, {
    'Fund join successful': (res) => res.status === 200,
    'Bad Request (400)': (res) => res.status === 400,
    'Unauthorized (401)': (res) => res.status === 401,
    'Internal Server Error (500)': (res) => res.status === 500,
    'Request Timeout (408)': (res) => res.status === 408,
  });
  console.log(fundRes.status);
  console.log(fundRes.body);

  // Get the current hour and minute
  const currentHour = dayjs().hour();
  const currentMinute = dayjs().minute();

  // If the current minute is 0 (i.e., a new hour started) and the hourCounter is less than 10
  if (currentMinute === 0 && hourCounter < 10) {
    // Purchase units
    const purchaseRequestReference = `znnl-${dayjs().format('DD/MM/YYYY').toUpperCase()}-${faker.random.number({ min: 100, max: 999999999999 })}`;
    const purchasePayload = {
      ClientIdentifier: investorPayload.ClientIdentifier,
      FundShareClassCode: "MMF",
      PurchaseValue: faker.random.number({ min: 1000, max: 100000 }),
      PurchaseRequestReference: purchaseRequestReference,
      PaymentModeCode: "MPESA",
      PaymentReference: faker.random.alphaNumeric(10)
    };

    const purchaseRes = http.post(purchaseUrl, JSON.stringify(purchasePayload), { headers: headers });
    check(purchaseRes, {
      'Purchase successful': (res) => res.status === 200,
      'Bad Request (400)': (res) => res.status === 400,
      'Unauthorized (401)': (res) => res.status === 401,
      'Internal Server Error (500)': (res) => res.status === 500,
      'Request Timeout (408)': (res) => res.status === 408,
    });
    console.log(purchaseRes.status);
    console.log(purchaseRes.body);

    // Fetch balance
    const balancePayload = {
      ClientIdentifier: investorPayload.ClientIdentifier
    };
    const balanceRes = http.post(balanceUrl, JSON.stringify(balancePayload), { headers: headers });
    console.log(balanceRes.status);
    console.log(balanceRes.body);

    // Check balance and make sale request
    const balance = JSON.parse(balanceRes.body);
    if (balance > 0) {
      const saleRequestReference = `S-${dayjs().format('DDMMMYYYY').toUpperCase()}-${faker.random.number({ min: 100, max: 999 })}`;

      const salePayload = {
        ClientIdentifier: investorPayload.ClientIdentifier,
        FundShareClassCode: "MMF",
        SaleValue: faker.random.number({ min: 1000, max: 100000 }),
        SaleRequestReference: saleRequestReference,
        PaymentModeCode: "MPESA"
      };

      const saleRes = http.post(saleUrl, JSON.stringify(salePayload), { headers: headers });
      check(saleRes, {
        'Sale successful': (res) => res.status === 200,
        'Bad Request (400)': (res) => res.status === 400,
        'Unauthorized (401)': (res) => res.status === 401,
        'Internal Server Error (500)': (res) => res.status === 500,
        'Request Timeout (408)': (res) => res.status === 408,
      });
      console.log(saleRes.status);
      console.log(saleRes.body);
    } else {
      console.log("Balance is not greater than 0. Skipping sale request.");
    }

    // Increment the hour counter
    hourCounter++;
  }

  // Add investor to investors array
  investors.push(investorPayload);

  // sleep for 5 minutes before running the test again
  // sleep(5 * 60);
}

export function handleSummary(data) {
  const csv = papaparse.unparse(investors);
  console.log(investors);
  const now = new Date();
  const options = {
    timeZone: 'Africa/Nairobi',
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
  };
  const kenyaTime = now.toLocaleString('en-US', options).replace(/[:/]/g, '-');
  const filename = `Regtojoin-Report_${kenyaTime}.html`;
  return {
    "investors_.csv": csv,
    [filename]: htmlReport(data),
    stdout: textSummary(data, { indent: " ", enableColors: true }),
  };
}

```

Also contains a config file where it fetches the defaults of the program before it runs.They defaults can be adjusted to suit the users demands.
```javascript
export default {
  endpoints: {
    authServerUrl: "https://cisapisecurity.innova.co.ke/core",
    investorUrl: "https://cisapi.innova.co.ke/api/Investors/Register",
    fundUrl: "https://cisapi.innova.co.ke/api/UnitTrust/JoinFund",
    purchaseUrl: "https://cisapi.innova.co.ke/api/UnitTrust/PurchaseUnitTrust",
    balanceUrl: "https://cisapi.innova.co.ke/api/UnitTrust/FetchInvestorUnitTrustFunds",
    saleUrl: "https://cisapi.innova.co.ke/api/UnitTrust/SellUnitTrust",
  },
  versions: {
    investor: "2.0",
    fund: "2.0",
    purchase: "3.0",
    balance: "2.0",
    sale: "3.0",
  },
  vus: 5,
  duration: "5h",
  token: {
    clientID: "CISUSSD",
    clientSecret: "tInSenElkshpuEZverE",
    scope: "innovaSuiteApi",
  },
};

```


The script uses K6 to simulate many users interacting with the UNIT TRUST API. It does things like signing up investors, joining funds, purchasing and selling units. To make things feel real, the script makes up data using the Faker library. It talks to the API using HTTP requests and keeps track of what the API says back to understand any issues. 

To try it out, save the script as a .js file and run it with "k6 run." You can set how long it runs in the options.

Bash
```Bash
k6 run regtojoin.js --summary-export=resultforregtojoin91.json --quiet > k6logfile 2>&1
```

#### The testing objectives for using K6 to test the UNIT TRUST API are:

* Request Rate Determination: The script is engineered to ascertain the maximum number of requests that can be effectively added per second. By systematically increasing the load and measuring response times, it aims to identify the threshold at which the system starts to exhibit performance degradation or errors.

* Load Capacity Assessment: Another key objective is to assess the API's ability to handle increasing volumes of data through requests. This is achieved by progressively scaling the amount of data in requests to evaluate the point at which the system becomes overwhelmed, potentially leading to breakdowns or slowdowns.

* Activity Evaluation: To provide a comprehensive assessment, the script generates a substantial level of activity. This activity is geared towards thoroughly evaluating the system's performance under realistic usage conditions, helping to identify bottlenecks, inefficiencies, or areas for improvement.

* Performance Evaluation: The code simulates a real-world scenario where multiple users interact with the investment platform simultaneously. By doing so, it assesses the platform's response times and overall performance under different levels of load.

* Scalability Assessment: The test scenarios are designed to evaluate how well the platform can scale to accommodate a growing number of concurrent users. This is important to ensure that the system can handle increased user demand without compromising its performance.

* Stress Testing: The code applies stress to the system by generating a significant number of requests and interactions. This helps identify the breaking points and limitations of the system, allowing for proactive measures to be taken to prevent crashes or failures.

* Concurrency Analysis: The code assesses the system's ability to handle a high number of concurrent connections. This is crucial to ensure that the platform's resources, such as database connections, are efficiently managed without causing contention.

* Error Detection: By checking HTTP response codes and patterns, the code identifies potential issues such as incorrect request handling, authentication failures, or server errors. Detecting and addressing these issues early is essential for maintaining the platform's reliability.

* Data Consistency: Through various interactions like purchases, balance checks, and sales, the code assesses the accuracy and consistency of data returned by the API. Any discrepancies could indicate underlying issues that need to be addressed.

* Reporting and Analysis: The code generates comprehensive reports, including CSV, HTML, and text summaries. These reports provide insights into the system's performance, potential bottlenecks, and areas for improvement. They also serve as documentation for the load testing process.

By achieving these objectives, the load testing enables the identification of potential performance bottlenecks, scalability limitations, and other issues that may impact the investment platform's users. Addressing these issues based on load testing results can enhance the platform's stability, user experience, and overall reliability.  
### HOW TO RUN THE SCRIPT FOR THE DESIRED OUTCOME  

To run the script and achieve the desired outcome, follow these steps:

1. **Install K6:**
K6 is a tool built for efficient load testing and supports JS. To use the script, you need to install K6 by Visit the K6 website: https://k6.io/open-source
Download the appropriate installer for your operating system (Windows, macOS, Linux).
Follow the installation instructions for your platform.


2. **Create a new file with the .js extension.**
Open your code editor, create a new file and give it a name with the .js extension.

3. **Copy the script and paste it into the new file.**
Copy the script provided and paste it into the new file. Ensure that you have copied the entire script.

4. **Install dependencies:**
The script relies on some dependencies that must be installed before you can run it. Install the following dependencies:



* k6/http 
* k6 
* k6-reporter 
* Faker 
* papaparse 
To install these dependencies, you can run the following command in your terminal:

```Bash
npm install k6/http
npm install k6-reporter
npm install faker
npm install papaparse
npm install k6-summary

```
This command will install all the required dependencies globally on your machine.

5. **Run the script:**
To run the script, open your terminal, navigate to the folder containing the script file, and run the following command:


```Bash
k6 run script.js
```
This command will execute the script and start load testing.

NOTE: Command to Generate Logs and Reports (JSON and HTML):

If you wish to generate comprehensive logs and reports, including both JSON and HTML formats, use the following command:
Bash
```Bash
k6 run regtojoin.js --summary-export=resultforregtojoin91.json --quiet > k6logfile 2>&1
```
This command executes the script, captures logs, and produces both JSON and HTML reports. The --summary-export flag specifies the JSON report file, while the --out flag with json=report.json and html=report.html generates both JSON and HTML reports. The --quiet flag ensures that the console output is minimized, and the > k6logfile 2>&1 part captures logs and saves them to a file named k6logfile.

By following these steps, you'll be able to effectively run your load testing script, obtain valuable logs and reports, and analyze the results for performance evaluation.

6. **View test results:**
After the script completes its run, you'll want to check out the results. The terminal will show you the status of the requests and the data they returned. Alongside this, the script generates two key files: an HTML report and a JSON file with important TEST metrics.

The HTML report, named something like regtojoinreport23_${kenyaTime}.html, carries a timestamp reflecting when it was generated in the Africa/Nairobi timezone (${kenyaTime}). This report goes beyond just numbers by visualizing the test results with charts and graphs. You can open this report in your web browser to easily grasp how everything performed.

These resources provide a human-friendly way to digest the load testing outcomes, making it simpler to understand and make well-informed decisions moving forward

7. **Adjust the test parameters:**
If you want to change the number of virtual users or the duration of the test, you can modify the options object in the config file "config.js". For example, to increase the number of virtual users to 200, you can change vus: 150 to vus: 200. Similarly, to increase the duration of the test to 6 hours, you can change duration: "5h" to duration: "6h".

Once you have made the changes, save the file, and run the script again using the command in step 5.

That's it! You now know how to run the provided K6 script and achieve the desired outcome.

### The Scope

Load testing tools are designed to simulate high traffic on an application and to identify performance issues. K6 is one such tool, which is designed to be a developer-centric load testing tool that is easy to use and can be integrated into the development workflow. It allows developers to write and execute performance tests in a familiar programming language and provides real-time metrics and analysis to help identify performance issues. K6 supports HTTP/1.1, HTTP/2, WebSocket, and TCP protocols and can be used to test a wide range of applications, including APIs, websites, and microservices.

In our case, we are using K6 to test the performance of a REST API for registering investors, joining funds,purchasing units and sale of units. However, K6 is not the only tool that can be used for load testing. Other tools such as NBomber can also be used to perform similar tests.

#### The Solution Overview:

The provided script is designed for comprehensive load testing using the K6 testing tool. It expertly simulates traffic across two crucial API endpoints, orchestrating four essential processes: registration, joining a fund, purchasing units, and selling units. To emulate real-world scenarios, the script dynamically generates randomized data, registering and enrolling investors into a unit trust fund. This meticulously crafted script is tailored to run over a five-hour span, orchestrating the interaction of 150 virtual users (VUs). Throughout the process, it meticulously captures and logs essential data on the registered investors. As the script concludes its operations, it crafts a comprehensive HTML report that clearly shows the outcomes, ensuring an insightful and human-friendly assessment of the load testing endeavor.

### Solution:

## Code Structure
##### Import Statements
The code begins by importing required modules, including k6's HTTP functions, sleep function, various external libraries like faker, PapaParse, and dayjs, as well as custom configuration (config.js) and reporting modules.

##### Token Management
The code defines functions for managing authentication tokens:

* getAccessToken():
 This function obtains an access token using the client credentials flow and stores it along with its expiration time.
* ensureAccessToken():
 This function ensures that a valid access token is available and refreshes it if it's about to expire.
##### Load Test Configuration
The code sets up the load test configuration using values from the config.js file.

##### Load Test Scenario
The primary load test scenario is defined within the default function exported by the module. The scenario includes the following steps:

* Token Management: The ensureAccessToken() function is called to ensure a valid access token is available.

* Investor Payload: An investor payload is generated using fake data from the faker library. This payload contains simulated investor information.

* Fund Payload: A fund payload is generated for the investor to join a fund. This payload includes relevant information for fund joining.

* HTTP Requests: The code sends HTTP requests to various endpoints of the investment platform's API using the generated payloads. It checks the responses for different HTTP status codes to validate the success of the requests.

* Hourly Logic: The code tracks the current hour and minute using dayjs. If the current minute is 0 and the hourCounter is less than 10, additional operations related to purchasing, checking balance, and initiating a sale are performed. This logic simulates specific actions that are performed once per hour.

* Investor Tracking: The investor's payload is added to the investors array, which keeps track of simulated investors.

* Sleep: A sleep period of 5 minutes is introduced before the load test scenario is executed again.

##### Summary Handling
The handleSummary(data) function is defined to handle the test summary. It performs the following tasks:

* CSV Generation: It generates a CSV representation of the investors array using the PapaParse library.

* Report Generation: It generates an HTML report using the k6-reporter module, which provides a comprehensive visualization of test results.

* Text Summary: It generates a text-based summary of the test results using the k6-summary module.

* Output: The function returns an object containing the CSV data, HTML report, and text summary, which are subsequently used for reporting.

K6 provides a solution to the challenges of load testing by providing a developer-centric tool that is easy to use and provides real-time metrics and analysis. Some of the key features of K6 include:

***STRONG HOLDS OF K6***

### Simple Scripting Language
K6 uses a simple, yet powerful, scripting language that is based on JavaScript. This allows developers to create complex test scenarios that simulate real-world user behavior, such as sending HTTP requests, waiting for responses, and extracting data from the responses.

#### Real-Time Metrics
K6 provides real-time metrics during test execution, including response time, throughput, and error rate. This allows developers to identify performance bottlenecks and make adjustments to improve application performance.

#### Integrations and Plugins
K6 provides integrations and plugins for a variety of tools, including Grafana, InfluxDB, and Jenkins. This allows developers to incorporate K6 into their existing development workflow and use their preferred tools for metrics and reporting.

#### Scalability
K6 is designed to be highly scalable and can be used to test applications running on a small development machine or a large cluster of servers. It supports distributed testing, which allows tests to be run across multiple machines to simulate high levels of traffic.

#### Open-Source
K6 is an open-source tool, which means that it is free to use and can be customized to meet the needs of individual projects. This can help reduce the cost of load testing and make it more accessible to developers and DevOps teams.

## SOME OF THE CHALLENGES ENCOUNTERED INCLUDE:

1. Generating unique data:
 The script uses the Faker library to generate random data for each investor. However, there is a chance that duplicate data could be generated, which could cause issues with the registration process or with analyzing the data afterwards.

1.  Handling errors:
The script makes API calls to register investors and join them to a fund. If there are any errors with these requests, the script may fail or generate incorrect data. It's important to handle errors and log any relevant information to help with troubleshooting.

1. Performance issues: The script is designed to simulate a high load on the system by registering and joining a large number of investors. This could cause performance issues or even overload the system being tested.


1. Saving data: The script generates data for each investor, but it does not save this data to a file by default. It's important to save this data to a file so that it can be analyzed later. One potential challenge here is deciding on a format for the data that will be useful for analysis, such as CSV or JSON.


1. Reporting: The script uses the k6-reporter library to generate an HTML report at the end of the test run. This could potentially fail if there are issues with the library or if the data being analyzed is too large to fit into memory. It's important to test the report generation and make sure it works correctly before relying on it for analysis.


#### Conclusion

In conclusion, automated testing is an essential part of the software development process, and it can help to improve the quality, reliability, and performance of software products. By following the steps outlined above, developers can ensure that their code is thoroughly tested and optimized for maximum efficiency and effectiveness.
The provided code demonstrates a comprehensive load testing scenario for an investment platform's API. It uses the k6 load testing tool to simulate user interactions, make HTTP requests, and analyze the platform's performance under various scenarios. The code is structured with a focus on token management, payload generation, HTTP requests, and result reporting. By utilizing fake data and predefined scenarios, this load testing approach aids in identifying potential bottlenecks and performance issues within the investment platform's API.


### Appreciation:

[![normanf](https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png)](https://www.buymeacoffee.com/normanf)

Thank you for being a part of this journey. Your encouragement means the world and keeps the passion alive. Here's to more learning and growth together!
