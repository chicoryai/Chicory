# Chicory AI

This project aims to analyze SparkSQL queries within a Spark Python project to provide optimization recommendations. It leverages Large Language Models (LLM) for analysis and real-time metadata from a database (DB) for context, such as Data Definition Language (DDL) and statistics. The goal is to enhance the efficiency and performance of SparkSQL queries by offering actionable insights.

## Onboarding

### Prerequisites
Before you begin, ensure you have:
- A GitHub account
- A project repository with SparkSQL queries (we support both *.sql and *.py)

### Step 1: Sign Up for the API Service
1. Visit https://chicory.ai/signup sign-up page.
2. Register for an account to receive your API key. Keep this key secure, as you'll need it to authenticate your requests.

### Step 2: Set Up the GitHub Workflow
1. Navigate to your project repository on GitHub.
2. Create a new directory named .github/workflows if it doesn't already exist.
3. Inside the workflows directory, add a file named optimize-workflow.yml. You can use the scaffold provided in the configuration files you cloned earlier.
```bash
# set workdir as repo head
mkdir -p .github/workflows && curl -fsSL https://raw.githubusercontent.com/TryCarbonara/Arabica/main/github.actions/optimize-workflow.yml -o .github/workflows/optimize-workflow.yml
```
Ensure you place these files in the appropriate directories within your project repository.

### Step 3: Add Dockerfile to .chicory/ Folder
1. In your project repository, create a directory named .chicory if it doesn't exist.
2. Add a Dockerfile to the .chicory directory. Follow the format specified in the configuration files to ensure compatibility with the workflow.
```bash
# set workdir as repo head
mkdir -p .chicory && curl -fsSL https://raw.githubusercontent.com/TryCarbonara/Arabica/main/github.actions/.chicory/Dockerfile -o .chicory/Dockerfile
```
Ensure you place these files in the appropriate directories within your project repository.

### Step 4: Configure Secrets
1. Go to your repository's Settings tab, then navigate to Secrets > Actions.
2. Create a new secret named CHICORY_TOKEN and paste your API key obtained during sign-up.
3. Create secrets named DB_HOST, DB_PORT, DB_USER, DB_PASS, DB_NAME to include details about the metadata service.

### [Optional] Step 5: Offline Metadata Server Configuration
To enhance the precision of data pipeline analysis, Chicory utilizes the context from your metadata service by understanding the backend setup. If direct access to the database is not feasible as outlined in Step 4.3, offline context provision is also supported:
* Database Schema (DDL) Information: To obtain DDL details of the database tables, use the command below. Store the resulting information in .chicory/stats.json. For a reference file, see: https://raw.githubusercontent.com/TryCarbonara/Arabica/main/github.actions/.chicory/stats.json
```bash
# PostgreSQL:
pg_dump -s -h [host] -U [username] -d [database_name] > db_schema.sql

# Redshift:
SELECT 'CREATE TABLE ' || tablename || ' (' || LISTAGG(column || ' ' || type, ', ') WITHIN GROUP (ORDER BY ordinal_position) || ');'
FROM (
    SELECT tablename, ordinal_position, column, type
    FROM pg_table_def
    WHERE schemaname = 'public' -- Specify your schema name if different
) AS dt
GROUP BY tablename;
```
* Database Statistics: To gather database statistics, execute the following command. The output should be saved in .chicory/ddl.json. For an example, visit: https://raw.githubusercontent.com/TryCarbonara/Arabica/main/github.actions/.chicory/ddl.json
```bash
# PostgreSQL:
SELECT relname AS "Table",
       n_live_tup AS "Live Rows",
       n_dead_tup AS "Dead Rows",
       last_vacuum AS "Last Vacuum",
       last_analyze AS "Last Analyze"
FROM pg_stat_user_tables;

# Redshift:
SELECT table_id, 
       table, 
       size, 
       pct_used, 
       empty, 
       unsorted, 
       stats_off 
FROM svv_table_info;
```

### Step 6: Trigger the Workflow
The workflow will automatically trigger on each Pull Request to the main branch of your repository. To manually test it, you can:
1. Make a change to a SparkSQL query within your project.
2. Commit and push the change to a new branch.
3. Open a Pull Request to merge the changes into the main branch.
The workflow will analyze the SparkSQL queries and provide optimization suggestions as comments on the Pull Request.

## Support
For support, please open an issue in the project repository or contact the service provider directly through their support channels.

## License
This project is licensed under the [LICENSE NAME]. See the LICENSE file in the repository for more details.
