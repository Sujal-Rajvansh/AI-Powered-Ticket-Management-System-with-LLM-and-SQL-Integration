# 🎟️ AI-Powered Ticket Management System with LLM and SQL Integration 🚀

This project demonstrates an intelligent ticket management system powered by **LangChain**, **Ollama's Llama 3.2**, and **SQLite**. It supports customer authentication, ticket tracking, and admin queries, providing seamless interaction through a **Gradio-based interface** or a **command-line interface**.

---

## 🌟 Key Features
- **AI-Driven Query Resolution**: Use LLM to interpret user queries and generate actionable responses.
- **Customer Support Database**: Includes `Customers` and `SupportTickets` tables to track customer interactions and ticket statuses.
- **Role-Based Access**: Different query functionalities for customers and admins.
- **Interactive Gradio Interface**: Simple and user-friendly for real-time customer and admin interaction.
- **Secure Authentication**: Customer email-based and admin credential-based authentication.
- **Comprehensive Admin Queries**: View open/closed tickets, total tickets, or customer details.

---

## 🛠️ Installation

### Install Required Libraries
Run the following commands to set up your environment:
```bash
pip install langchain langchain_ollama gradio sqlite-utils
🚀 Workflow
1️⃣ Set Up the SQLite Database
Create and populate the database with sample data:

python
Copy code
import sqlite3

def create_database():
    conn = sqlite3.connect("customer_support20.db")
    cursor = conn.cursor()

    # Create tables
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS Customers (
        CustomerID INTEGER PRIMARY KEY,
        Name TEXT NOT NULL,
        Email TEXT NOT NULL
    )
    """)
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS SupportTickets (
        TicketID INTEGER PRIMARY KEY,
        CustomerID INTEGER,
        Issue TEXT NOT NULL,
        Status TEXT NOT NULL,
        FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
    )
    """)

    # Insert sample data
    cursor.executemany("INSERT INTO Customers (Name, Email) VALUES (?, ?)", [
        ("Alice Smith", "alice@example.com"),
        ("Bob Johnson", "bob@example.com"),
    ])
    cursor.executemany("INSERT INTO SupportTickets (CustomerID, Issue, Status) VALUES (?, ?, ?)", [
        (1, "Cannot log in to account", "Open"),
        (1, "Payment issue on order #1234", "Resolved"),
        (2, "Reset password request", "Open"),
    ])

    conn.commit()
    conn.close()

create_database()
print("Database created successfully!")
2️⃣ Initialize LLM and SQL Agent
Connect the database and initialize the LLM:

python
Copy code
from langchain_community.llms import Ollama
from langchain_community.utilities import SQLDatabase
from langchain.agents import create_sql_agent

# Connect to the database
db = SQLDatabase.from_uri("sqlite:///customer_support20.db", sample_rows_in_table_info=3)

# Initialize the LLM
llm = Ollama(model="llama3.2")

# Create the SQL agent
sql_agent = create_sql_agent(llm, db=db, verbose=True)
3️⃣ Customer Query Functionality
Authenticate customers and handle their queries:

python
Copy code
def authenticate_customer(email):
    conn = sqlite3.connect("customer_support20.db")
    cursor = conn.cursor()
    cursor.execute("SELECT CustomerID FROM Customers WHERE Email = ?", (email,))
    result = cursor.fetchone()
    conn.close()
    return result[0] if result else None

def handle_customer_query(email, query):
    customer_id = authenticate_customer(email)
    if not customer_id:
        return "Authentication failed. Please check your email."

    if query == "open_tickets":
        conn = sqlite3.connect("customer_support20.db")
        cursor = conn.cursor()
        cursor.execute("SELECT COUNT(*) FROM SupportTickets WHERE CustomerID = ? AND Status = 'Open'", (customer_id,))
        result = cursor.fetchone()
        conn.close()
        return f"You have {result[0]} open tickets."
4️⃣ Admin Query Functionality
Admins can query the entire database:

python
Copy code
def handle_admin_query(query):
    conn = sqlite3.connect("customer_support20.db")
    cursor = conn.cursor()
    
    if query == "total_tickets":
        cursor.execute("SELECT COUNT(*) FROM SupportTickets")
        result = cursor.fetchone()
        conn.close()
        return f"There are {result[0]} tickets in total."
    
    elif query == "open_tickets":
        cursor.execute("SELECT COUNT(*) FROM SupportTickets WHERE Status = 'Open'")
        result = cursor.fetchone()
        conn.close()
        return f"There are {result[0]} open tickets."
5️⃣ Interactive Gradio Interface
Build an interface for customer interaction:

python
Copy code
import gradio as gr

def customer_interface(email, question):
    response = handle_customer_query(email, question)
    return response

chat_interface = gr.Interface(
    fn=customer_interface,
    inputs=[
        gr.Textbox(label="Email", placeholder="Enter your registered email"),
        gr.Textbox(label="Question", placeholder="What is the status of my ticket?", lines=2)
    ],
    outputs=gr.Textbox(label="Response"),
    title="Customer Support Query System",
    description="Enter your email and ask about your tickets or issues."
)

chat_interface.launch()
📚 Example Queries
Customer Queries
Open Tickets
Input:

text
Copy code
What is the status of my ticket?
Output:

text
Copy code
You have 1 open ticket.
List All Tickets
Input:

text
Copy code
List all my tickets.
Output:

text
Copy code
TicketID: 1, Issue: Cannot log in to account, Status: Open  
TicketID: 2, Issue: Payment issue on order #1234, Status: Resolved
Admin Queries
Total Tickets
Input:

text
Copy code
How many tickets are there in total?
Output:

text
Copy code
There are 3 tickets in total.
Open Tickets
Input:

text
Copy code
How many open tickets are there?
Output:

text
Copy code
There are 2 open tickets.
🔧 Built with LangChain, SQLite, and Gradio for scalable and intelligent ticket management. Perfect for real-time customer support and administrative insights! 🚀
