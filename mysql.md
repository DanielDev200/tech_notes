# MySQL

## Overview of What We Tried, What Worked, and Why It Worked

### Problem Summary
Your Node.js application was not picking up the values from the .env file, and instead, system-level environment variables (e.g., MYSQL_USER, MYSQL_PASSWORD) were being used.

### Steps We Took

Confirming the .env File Path - We verified where the .env file needed to be relative to the config.js file.
So lution: We explicitly set the .env path in config.js using:

``
require('dotenv').config({ path: require('path').resolve(__dirname, '../.env') });
``

This made dotenv look for the .env file one directory up from the db directory.

### Why it didn’t fully work:
While this loaded the file correctly, the environment variables were still overridden by global/system variables.

### Debugging Variable Conflicts

We logged the resolved path and process.env to confirm:
1. The correct .env file path was found.
2. System-level MYSQL_* variables were taking precedence

### Forcing dotenv to Override System Variables
We modified the dotenv call to include the override option:

``
require('dotenv').config({ 
    path: require('path').resolve(__dirname, '../.env'),
    override: true
});
``

This explicitly forced dotenv to overwrite any pre-existing environment variables, even those set globally.

### What Worked
The override: true option ensured that variables from your .env file took precedence over system-level environment variables.

### Why It Worked
By default, dotenv does not overwrite existing environment variables.

System or global environment variables (e.g., MYSQL_USER, MYSQL_PASSWORD) were being prioritized over those in the .env file.
Adding override: true instructed dotenv to replace any pre-existing values with those from the .env file.

### Key Takeaways
1. Path Matters: dotenv requires an accurate path to locate the .env file.
2. Environment Variable Precedence: System or globally set environment variables take precedence unless explicitly overwritten.
3. Force Override: Using override: true ensures .env values are prioritized.
