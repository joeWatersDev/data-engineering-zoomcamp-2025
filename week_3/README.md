# DATA WAREHOUSE

### OLTP VS OLAP

Online transaction processing (OLTP) and Online analytical processing (OLAP) are two different database paradigms. This is how they compare:

|          |   OLTP   |   OLAP   |
| -------- | -------- | -------- |
| Purpose  | Control and run essential business operations in real time | Plan, solve problems, support decisions, discover hidden insights |
| Data Updates  | Short, fast updates initiated by user | Data periodically refreshed with scheduled, long-running batch jobs |
| Database design  | Normalized databases for efficiency | Denormalized databases for analysis |
| Space Requirements  | Generally small if historical data is archived | Generally large due to aggregating large datasets |
| Fxamples  | Customer-facing personnel, clerks, online shoppers | Knowledge workers such as data analysts, business analysts, and executives |

A *data warehouse* is an **OLAP** solution largely used for reporting and data analysis.

### BigQuery

BigQuery is Google's data warehouse tool that is serverless - it has no servers to manage or database software to install. It also has built in features like machine learning and business intelligence.