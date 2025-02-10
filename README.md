# Full Stack App Using Docker 🚀🐳

## Overview  
This project demonstrates how to create a **full-stack application** using Docker, where two containers—**PostgreSQL** (database) and **Streamlit** (web app)—communicate within a **Docker network**. We’ll set up a **custom Docker network**, run a **PostgreSQL database** container, and connect it to a **Streamlit app** container, providing a simple interface for database interaction.

## 📚 Documentation & Prerequisites  
Ensure you have the following installed:  
- [Docker](https://www.docker.com/)  
- [Docker Desktop](https://www.docker.com/products/docker-desktop)  
- [Docker Hub Account](https://hub.docker.com/)  

Also, make sure to have the following in your project directory:  
- **Streamlit App (stream.py)**
- **requirements.txt** (including `streamlit` and `psycopg2` for PostgreSQL)

## 🏗️ Setup & Configuration  

### **1️⃣ Create a Custom Docker Network**  
Start by creating a custom Docker network to allow the containers to communicate:  
```sh
docker network create my_network
```

### **2️⃣ Set Up the PostgreSQL Database Container**  
Run a PostgreSQL container that will be connected to the `my_network` network:  
```sh
docker run -d \
  --name my_postgres \
  --network my_network \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=adminpassword \
  -e POSTGRES_DB=mydb \
  postgres
```

### **3️⃣ Create the Streamlit App Container**  
Now, create a Dockerfile for your **Streamlit app** container. Below is a sample Dockerfile:

```Dockerfile
# Use official Python image
FROM python:3.9

# Set working directory
WORKDIR /app

# Copy app files to the container
COPY . .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Expose Streamlit port
EXPOSE 8501

# Command to run Streamlit app
CMD ["streamlit", "run", "stream.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

Make sure your **`requirements.txt`** includes:

```txt
streamlit
psycopg2
```

### **4️⃣ Build and Run the Streamlit App Container**  
Build the Docker image for the Streamlit app and run the container connected to `my_network`:

1. **Build the image**:  
   ```sh
   docker build -t my_streamlit_app .
   ```

2. **Run the container**:  
   ```sh
   docker run -d \
     --name streamlit_app \
     --network my_network \
     -p 8501:8501 \
     my_streamlit_app
   ```

### **5️⃣ Connect Streamlit App to PostgreSQL**  
In the `stream.py` file, connect the Streamlit app to PostgreSQL by using the container name (`my_postgres`) as the **host**.

Here’s a sample `stream.py`:

```python
import streamlit as st
import psycopg2

# Connect to PostgreSQL database
conn = psycopg2.connect(
    dbname="mydb",
    user="admin",
    password="adminpassword",
    host="my_postgres",  # Container name as hostname
    port="5432"
)
cur = conn.cursor()

# Query to check DB version
cur.execute("SELECT version();")
db_version = cur.fetchone()

st.title("Streamlit App with PostgreSQL")
st.write("Connected to database:", db_version)

# Close the connection
cur.close()
conn.close()
```

### **6️⃣ Test the Setup**  
Open a browser and go to [http://localhost:8501](http://localhost:8501) to view the Streamlit app and check if it’s connected to the database.

### **7️⃣ Create and Insert Dummy Data into PostgreSQL**  
To test the database, connect to the PostgreSQL container and insert some dummy data:

1. **Access PostgreSQL**:
   ```sh
   docker exec -it my_postgres psql -U admin -d mydb
   ```

2. **Create a sample table and insert data**:
   ```sql
   CREATE TABLE users (
       id SERIAL PRIMARY KEY,
       name VARCHAR(100),
       email VARCHAR(100) UNIQUE
   );

   INSERT INTO users (name, email) VALUES
   ('Alice Johnson', 'alice@example.com'),
   ('Bob Smith', 'bob@example.com'),
   ('Charlie Brown', 'charlie@example.com');

   SELECT * FROM users;
   ```

### **8️⃣ Create Custom Bridge Network**  
Optionally, create a **custom bridge network** for additional isolation or performance:  
```sh
docker network create --driver bridge my_custom_network
```

## 🎯 Conclusion  
You've now created a **full-stack app** using **Docker**, connecting a **PostgreSQL database** with a **Streamlit app** in a Docker containerized environment. This setup can be extended to include more complex backend logic or additional frontend features.

---
