# Security-Information-and-Event-Management-SIEM-System

To create a fully functional SIEM system with the steps you've mentioned (collect logs, parse logs, store logs, detect alerts, and visualize logs), here's a detailed breakdown of how you can structure the code into a modular project.

### Step-by-Step Project Creation

#### Folder Structure
Here's the folder structure that will help organize the project efficiently:

```
siem_project/
│
├── main.py                # Entry point of the SIEM system
├── log_collector.py       # Module for collecting logs
├── log_parser.py          # Module for parsing logs
├── db_manager.py          # Module for database setup and storing logs
├── alert_manager.py       # Module for detecting alerts
├── visualizer.py          # Module for visualizing logs
└── siem_logs.db           # SQLite database for storing logs
```

### 1. **Create the `main.py` File**
This file will serve as the entry point to your SIEM system, orchestrating all the steps:

```python
# from log_collector import collect_logs
from log_parser import parse_logs
from db_manager import setup_database, store_logs
from alert_manager import detect_alerts
from visualizer import visualize_logs

def main():
    print("Starting the SIEM system...")
    
    # Step 1: Setup Database
    setup_database()
    
    # Step 2: Collect Logs
    logs = collect_logs()
    print(f"Collected {len(logs)} logs.")
    
    # Step 3: Parse Logs
    parsed_logs = parse_logs(logs)
    print(f"Parsed {len(parsed_logs)} logs.")
    
    # Check if parsed logs are valid
    if parsed_logs:
        print("Parsed logs successfully.")
    else:
        print("No logs parsed, check your log format.")
    
    # Step 4: Store Logs
    store_logs(parsed_logs)
    print("Logs stored in database.")
    
    # Step 5: Detect Alerts
    alerts = detect_alerts(parsed_logs)
    for alert in alerts:
        print(alert)
    
    # Step 6: Visualize Logs
    visualize_logs(parsed_logs)

if __name__ == "__main__":
    main()
```

### 2. **Create the `log_collector.py` File**
This module will handle the collection of logs (either from files, network devices, or external sources):

```python
# log_collector.py
import os

def collect_logs():
    log_file_path = r"C:\Users\Michael M\Downloads\siem_project\logfile.log"  # Use raw string
    logs = []
    try:
        with open(log_file_path, 'r') as file:
            logs = file.readlines()
    except FileNotFoundError:
        print(f"Log file {log_file_path} not found!")
    return logs
```

### 3. **Create the `log_parser.py` File**
This module will handle parsing the raw logs into structured data:

```python
# def parse_logs(logs):
    parsed_logs = []
    for log in logs:
        print(f"Parsing log: {log}")  # Debug print
        parts = log.strip().split(' ')
        if len(parts) >= 3:
            timestamp = parts[0] + ' ' + parts[1]
            log_level = parts[2]
            message = ' '.join(parts[3:]) if len(parts) > 3 else ''
            parsed_logs.append({
                'timestamp': timestamp,
                'log_level': log_level,
                'message': message
            })
        else:
            print(f"Failed to parse log: {log}")  # Debug print for logs that don't meet the criteria
    return parsed_logs
```

### 4. **Create the `db_manager.py` File**
This module will handle setting up the SQLite database and storing parsed logs:

```python
# import sqlite3

def setup_database(db_name="siem_logs.db"):
    """Setup the database with the necessary tables."""
    try:
        with sqlite3.connect(db_name) as conn:
            cursor = conn.cursor()
            cursor.execute('''
                DROP TABLE IF EXISTS logs
            ''')
            cursor.execute('''
                CREATE TABLE logs (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    timestamp TEXT,
                    level TEXT,
                    message TEXT
                )
            ''')
            conn.commit()
            print("Database setup complete.")
    except sqlite3.Error as e:
        print(f"Database setup failed: {e}")

def store_logs(parsed_logs, db_name="siem_logs.db"):
    """Store parsed logs in the database."""
    try:
        with sqlite3.connect(db_name) as conn:
            cursor = conn.cursor()
            for log in parsed_logs:
                cursor.execute("INSERT INTO logs (timestamp, level, message) VALUES (?, ?, ?)",
                               (log['timestamp'], log['log_level'], log['message']))
            conn.commit()
            print(f"Stored {len(parsed_logs)} logs in the database.")
    except sqlite3.Error as e:
        print(f"Failed to store logs: {e}")
```

### 5. **Create the `alert_manager.py` File**
This module will handle detecting alerts based on parsed logs:

```python
# alert_manager.py
def detect_alerts(parsed_logs):
    """Detect security alerts from parsed logs."""
    alerts = []
    
    for log in parsed_logs:
        if "failed login" in log['message'].lower():
            alerts.append(f"ALERT: Failed login attempt detected at {log['timestamp']}")
        if "error" in log['level'].lower():
            alerts.append(f"ALERT: Error log detected at {log['timestamp']}: {log['message']}")
    
    return alerts
```

### 6. **Create the `visualizer.py` File**
This module will handle visualizing the parsed logs, e.g., by printing them or creating graphs:

```python
# # visualizer.py
import matplotlib.pyplot as plt

def visualize_logs(parsed_logs):
    """
    Visualize logs in the form of a pie chart showing log level distribution.
    Handles empty or invalid data gracefully.
    """
    # Count log levels
    log_levels = {"INFO": 0, "ERROR": 0, "FAILED LOGIN": 0}
    
    # Count occurrences
    for log in parsed_logs:
        if log:
            if "failed login" in log['message'].lower():
                log_levels["FAILED LOGIN"] += 1
            elif log['level'].upper() == "ERROR":
                log_levels["ERROR"] += 1
            else:
                log_levels["INFO"] += 1
    
    # Only create the pie chart if there's data to plot
    if sum(log_levels.values()) > 0:
        labels = log_levels.keys()
        sizes = log_levels.values()
        
        plt.pie(sizes, labels=labels, autopct="%1.1f%%", startangle=90)
        plt.title("Log Level Distribution")
        plt.axis("equal")  # Equal aspect ratio ensures that pie is drawn as a circle.
        plt.show()
    else:
        print("No valid log data to visualize.")
```

### 7. **Setting Up the Database**
Before running the project, you need to make sure your SQLite database (`siem_logs.db`) is properly set up. When you run `main.py`, the `setup_database()` function will create the necessary tables in the database.

### 8. **Running the Project**
To run the project, navigate to the project directory and run:

```bash
python main.py
```

### Expected Output
When you run `main.py`, the output should look something like this:

```
Starting the SIEM system...
Database setup complete.
Collected 5 logs.
Parsed 5 logs.
Parsed logs successfully.
Stored 5 logs in the database.
ALERT: Failed login attempt at 2023-10-01 12:05:00
ALERT: Account locked at 2023-10-01 12:15:00
Visualizing logs:
2023-10-01 12:00:00 | INFO | User logged in
2023-10-01 12:05:00 | ERROR | Failed password attempt
2023-10-01 12:10:00 | INFO | User logged out
2023-10-01 12:15:00 | ERROR | Account locked due to multiple failed attempts
2023-10-01 12:20:00 | WARNING | Suspicious IP detected
```

### Conclusion

This setup organizes your SIEM system into modular components, each handling a specific task. You can easily extend the system by adding more log sources, more complex parsing logic, or even integrating external alerting systems (like email or SMS).
