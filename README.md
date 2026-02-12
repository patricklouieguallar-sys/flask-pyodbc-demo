# flask-pyodbc-demo (env vars are required)

A simple RESTful API built with Flask and SQL Server using pyodbc. This project demonstrates basic backend concepts such as database connectivity, CRUD style endpoints, structured logging, and error handling.

# Features :
# REST API built with Flask
# SQL Server integration via ODBC
# Insert and retrieve user records
# Structured logging
# JSON request/response handling
# Error handling with HTTP status codes

# Tech Stack :
# Python
# Flask
# SQL Server
# pyodbc
# Logging module

from flask import Flask, request, jsonify
import pyodbc
import logging
import os

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(message)s"
)

app = Flask(__name__)

# This function connects Python to MSSQL
def get_db_connection():
    try:
        logging.info("Connecting to database")

        conn = pyodbc.connect(
          "DRIVER={ODBC Driver 17 for SQL Server};"
          f"SERVER={os.getenv('DB_SERVER')};"
          f"DATABASE={os.getenv('DB_NAME')};"
          "trusted_connection=yes;"
        )
        return conn
    
    except Exception as e:
        logging.error(f"Connection failed {e}")
        raise

# Client sends JSON here to insert data into database
@app.route("/add_user", methods=["POST"]) #POST Client is send the data
def add_user():
    try:
        data = request.get_json()
        if not data:
            return jsonify({"error": "NO JSON RECIEVED"}), 400
        
        username = data.get("username")
        age = data.get("age")
       
        if not username or age is None:
           return jsonify({"Error":"Missing data's"}), 400

        conn = get_db_connection()
        cursor = conn.cursor()

        logging.info("Inserting user into database")
        cursor.execute(
            "INSERT INTO Users (Username, Age) VALUES (?, ?)",
            (username, age)
        )

        #save cahnges
        conn.commit()
        conn.close()
        return jsonify({
            "status":"success"
        }), 201
    except Exception as e:
        logging.error(f"Inserting failed: {e}")
        return jsonify({"error": "Internal server error"}), 500

# Client requests data from database
@app.route("/users",methods=["GET"]) #GET client wants data
def get_users():
    try:
        logging.info("Fetching data's")

        conn = get_db_connection()
        cursor = conn.cursor()

        cursor.execute("SELECT Username, Age FROM Users")
        rows = cursor.fetchall()

        users = []
        for row in rows:
            users.append({
                "username": row[0],
                "age": row[1]
            })

        conn.close()

        return jsonify(users), 200

    except Exception as e:
        logging.error(f"Fetching failed: {e}")
        return jsonify({"error": "Internal server error"}), 500

# This runs the HTTP server
if __name__ == "__main__":
    app.run(debug=True)
