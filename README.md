# fitness_tracker_frontend_24352
his is a brief, optional description of the project. A good description would be something like, "A database-driven fitness tracker application with PostgreSQL backend and web frontend."
Flask
psycopg2-binary
import psycopg2
import os

def get_db_connection():
    conn = psycopg2.connect(
        host=os.environ.get("DB_HOST", "localhost"),
        database=os.environ.get("DB_NAME", "fitness_db"),
        user=os.environ.get("DB_USER", "your_username"),
        password=os.environ.get("DB_PASS", "your_password")
    )
    return conn
    from flask import Flask, request, jsonify, render_template
from db_config import get_db_connection
import uuid

app = Flask(__name__)

# --- FRONTEND (Python handles serving the HTML page) ---
@app.route('/')
def index():
    """Renders the main frontend page."""
    return render_template('index.html')

# --- BACKEND (Python handles the API logic) ---
@app.route('/users', methods=['POST'])
def create_user():
    data = request.json
    try:
        conn = get_db_connection()
        cur = conn.cursor()
        cur.execute(
            "INSERT INTO users (username, email, password_hash) VALUES (%s, %s, %s) RETURNING user_id;",
            (data['username'], data['email'], data['password_hash'])
        )
        user_id = cur.fetchone()[0]
        conn.commit()
        cur.close()
        conn.close()
        return jsonify({"message": "User created successfully!", "user_id": str(user_id)}), 201
    except Exception as e:
        return jsonify({"error": str(e)}), 400

@app.route('/workouts', methods=['POST'])
def log_workout():
    data = request.json
    try:
        conn = get_db_connection()
        cur = conn.cursor()
        
        cur.execute("BEGIN;")

        cur.execute(
            "INSERT INTO workouts (user_id, workout_date, duration_minutes, workout_type) VALUES (%s, %s, %s, %s) RETURNING workout_id;",
            (uuid.UUID(data['user_id']), data['workout_date'], data['duration'], data['type'])
        )
        workout_id = cur.fetchone()[0]

        for ex in data['exercises']:
            cur.execute(
                "INSERT INTO exercises (workout_id, exercise_name, reps, sets, weight_kg) VALUES (%s, %s, %s, %s, %s);",
                (workout_id, ex['name'], ex['reps'], ex['sets'], ex['weight'])
            )

        cur.execute("COMMIT;")
        cur.close()
        conn.close()
        return jsonify({"message": "Workout logged successfully!", "workout_id": str(workout_id)}), 201
    except Exception as e:
        conn.rollback()
        return jsonify({"error": str(e)}), 400

@app.route('/workouts/<user_id>', methods=['GET'])
def get_user_workouts(user_id):
    try:
        conn = get_db_connection()
        cur = conn.cursor()
        cur.execute("SELECT * FROM workouts WHERE user_id = %s ORDER BY workout_date DESC;", (uuid.UUID(user_id),))
        workouts = cur.fetchall()
        cur.close()
        conn.close()
        return jsonify(workouts), 200
    except Exception as e:
        return jsonify({"error": str(e)}), 404

if __name__ == '__main__':
    app.run(debug=True)
    <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fitness Tracker</title>
</head>
<body>
    <div class="container">
        <h1>Log a New Workout</h1>
        <form id="log-workout-form">
            <label for="userId">User ID:</label>
            <input type="text" id="userId" required>
            
            <label for="workoutType">Workout Type:</label>
            <input type="text" id="workoutType" required>

            <label for="duration">Duration (minutes):</label>
            <input type="number" id="duration" required>

            <h3>Exercises</h3>
            <div id="exercises-container">
                <div class="exercise-item">
                    <input type="text" class="exercise-name" placeholder="Exercise Name" required>
                    <input type="number" class="reps" placeholder="Reps">
                    <input type="number" class="sets" placeholder="Sets">
                    <input type="number" class="weight" placeholder="Weight (kg)">
                </div>
            </div>
            <button type="button" id="add-exercise-btn">Add Another Exercise</button>
            <br>
            <button type="submit">Log Workout</button>
        </form>
        <hr>
        <h2>My Workouts</h2>
        <button id="view-workouts-btn">View My Workouts</button>
        <div id="workout-list"></div>
    </div>
    
    <script>
        // All the JavaScript from the previous response goes here
        document.getElementById('add-exercise-btn').addEventListener('click', () => {
            const container = document.getElementById('exercises-container');
            const newItem = document.createElement('div');
            newItem.className = 'exercise-item';
            newItem.innerHTML = `
                <input type="text" class="exercise-name" placeholder="Exercise Name" required>
                <input type="number" class="reps" placeholder="Reps">
                <input type="number" class="sets" placeholder="Sets">
                <input type="number" class="weight" placeholder="Weight (kg)">
            `;
            container.appendChild(newItem);
        });

        document.getElementById('log-workout-form').addEventListener('submit', async (e) => {
            e.preventDefault();

            const userId = document.getElementById('userId').value;
            const workoutType = document.getElementById('workoutType').value;
            const duration = document.getElementById('duration').value;
            const exercises = [];
            
            document.querySelectorAll('.exercise-item').forEach(item => {
                exercises.push({
                    name: item.querySelector('.exercise-name').value,
                    reps: item.querySelector('.reps').value,
                    sets: item.querySelector('.sets').value,
                    weight: item.querySelector('.weight').value,
                });
            });

            const workoutData = {
                user_id: userId,
                workout_date: new Date().toISOString().split('T')[0],
                duration: parseInt(duration),
                type: workoutType,
                exercises: exercises
            };

            try {
                const response = await fetch('/workouts', {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify(workoutData),
                });

                const result = await response.json();
                if (response.ok) {
                    alert('Workout logged successfully! ID: ' + result.workout_id);
                } else {
                    alert('Error logging workout: ' + result.error);
                }
            } catch (error) {
                console.error('Network error:', error);
            }
        });

        document.getElementById('view-workouts-btn').addEventListener('click', async () => {
            const userId = document.getElementById('userId').value;
            const workoutList = document.getElementById('workout-list');
            workoutList.innerHTML = 'Loading...';

            try {
                const response = await fetch(`/workouts/${userId}`);
                const workouts = await response.json();
                
                if (response.ok) {
                    workoutList.innerHTML = '';
                    workouts.forEach(w => {
                        const workoutElement = document.createElement('div');
                        workoutElement.innerHTML = `
                            <h4>Workout on ${w[2]}</h4>
                            <p>Type: ${w[4]}, Duration: ${w[3]} minutes</p>
                        `;
                        workoutList.appendChild(workoutElement);
                    });
                } else {
                    workoutList.innerHTML = 'No workouts found for this user.';
                }
            } catch (error) {
                console.error('Network error:', error);
            }
        });
    </script>
</body>
</html>
