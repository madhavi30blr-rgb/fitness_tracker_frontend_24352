# fitness_tracker_frontend_24352
his is a brief, optional description of the project. A good description would be something like, "A database-driven fitness tracker application with PostgreSQL backend and web frontend."
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fitness Tracker</title>
    <link rel="stylesheet" href="style.css">
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
    <script src="app.js"></script>
</body>
</html>
body {
    font-family: sans-serif;
    background-color: #f4f4f4;
    padding: 20px;
}
.container {
    max-width: 600px;
    margin: auto;
    background: white;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}
input[type="text"], input[type="number"] {
    width: 100%;
    padding: 8px;
    margin: 5px 0 10px 0;
    box-sizing: border-box;
}
button {
    padding: 10px 15px;
    cursor: pointer;
}
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
        const response = await fetch('http://127.0.0.1:5000/workouts', {
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
        const response = await fetch(`http://127.0.0.1:5000/workouts/${userId}`);
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
