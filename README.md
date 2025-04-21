//# Task 4
//This repo contains all the data necessary to get started on task 4, good luck!
import sqlite3
import pandas as pd

# Database connection
conn = sqlite3.connect("data_pipeline.db")
cursor = conn.cursor()

# Create tables (if not already created)
cursor.execute("""
CREATE TABLE IF NOT EXISTS Product (
    product_id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT,
    manufacturer TEXT,
    category TEXT,
    attributes TEXT
)""")

cursor.execute("""
CREATE TABLE IF NOT EXISTS Shipment (
    shipment_id INTEGER PRIMARY KEY AUTOINCREMENT,
    shipping_identifier TEXT UNIQUE,
    origin TEXT,
    destination TEXT
)""")

cursor.execute("""
CREATE TABLE IF NOT EXISTS Shipment_Product (
    shipment_id INTEGER,
    product_id INTEGER,
    quantity INTEGER,
    FOREIGN KEY (shipment_id) REFERENCES Shipment(shipment_id),
    FOREIGN KEY (product_id) REFERENCES Product(product_id)
)""")

# Load spreadsheet 0
df0 = pd.read_csv("spreadsheet0.csv")
df0.to_sql("Product", conn, if_exists="append", index=False)

# Load spreadsheets 1 & 2
df1 = pd.read_csv("spreadsheet1.csv")
df2 = pd.read_csv("spreadsheet2.csv")

# Merge shipment details
shipments = df1.merge(df2, on="shipping_identifier")

# Insert shipments into the database
for _, row in shipments.iterrows():
    cursor.execute("INSERT INTO Shipment (shipping_identifier, origin, destination) VALUES (?, ?, ?)", 
                   (row["shipping_identifier"], row["origin"], row["destination"]))
    shipment_id = cursor.lastrowid

    # Insert products linked to the shipment
    cursor.execute("INSERT INTO Shipment_Product (shipment_id, product_id, quantity) VALUES (?, ?, ?)", 
                   (shipment_id, row["product_id"], row["quantity"]))

# Commit changes and close connection
conn.commit()
conn.close()

print("Data successfully inserted into SQLite!") 
