from flask import Flask, render_template, request, redirect
import sqlite3

app = Flask(__name__)

# Function to initialize the database
def init_db():
    conn = sqlite3.connect('userdata.db')
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS users 
                 (id INTEGER PRIMARY KEY AUTOINCREMENT,
                  name TEXT,
                  email TEXT,
                  age INTEGER,
                  dob DATE)''')
    conn.commit()
    conn.close()

# Initialize the database
init_db()

# Route for the form page
@app.route('/')
def form():
    return render_template('form.html')

# Route for form submission
@app.route('/submit', methods=['POST'])
def submit():
    name = request.form['name']
    email = request.form['email']
    age = request.form['age']
    dob = request.form['dob']

    # Validate age as positive integer
    try:
        age = int(age)
        if age < 0:
            raise ValueError
    except ValueError:
        return "Age must be a positive integer"

    # Validation for email format
    if not ('@' in email and '.' in email):
        return "Invalid email format"

    conn = sqlite3.connect('userdata.db')
    c = conn.cursor()
    c.execute("INSERT INTO users (name, email, age, dob) VALUES (?, ?, ?, ?)", (name, email, age, dob))
    conn.commit()
    conn.close()

    return redirect('/users')

# Route for displaying user data
@app.route('/users')
def users():
    conn = sqlite3.connect('userdata.db')
    c = conn.cursor()
    c.execute("SELECT * FROM users")
    users = c.fetchall()
    conn.close()
    return render_template('users.html', users=users)

if __name__ == '__main__':
    app.run(debug=True)
