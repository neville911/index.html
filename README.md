<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>Aesthetic Protocol</title>
    <style>
        :root {
            --bg-color: #F8F9FA;
            --text-main: #121212;
            --text-muted: #6C757D;
            --card-bg: #FFFFFF;
            --accent: #20C997;
            --dark-card: #121212;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; }
        
        body { background-color: var(--bg-color); color: var(--text-main); padding: 20px; padding-top: 50px; }

        .header { margin-bottom: 25px; }
        .header h1 { font-size: 24px; font-weight: 800; }
        .header p { color: var(--text-muted); font-size: 14px; font-weight: 600; margin-bottom: 5px; }

        .progress-card { background-color: var(--dark-card); color: white; border-radius: 20px; padding: 20px; display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; }
        .progress-title { font-size: 16px; font-weight: 600; }
        .progress-sub { color: #ADB5BD; font-size: 12px; margin-top: 4px; }
        .ring { width: 50px; height: 50px; border-radius: 50%; border: 4px solid var(--accent); display: flex; justify-content: center; align-items: center; font-weight: bold; font-size: 14px; }

        .toggle-container { display: flex; background: #E9ECEF; border-radius: 12px; padding: 5px; margin-bottom: 25px; }
        .toggle-btn { flex: 1; padding: 12px; text-align: center; border-radius: 10px; font-size: 14px; font-weight: 600; color: var(--text-muted); cursor: pointer; transition: 0.2s; }
        .toggle-btn.active { background: var(--dark-card); color: white; }

        .section-title { font-size: 18px; font-weight: 700; margin-bottom: 15px; }

        .activity-card { background: var(--card-bg); border-radius: 15px; padding: 15px; display: flex; align-items: center; margin-bottom: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.02); }
        .activity-icon { width: 45px; height: 45px; border-radius: 10px; background-color: var(--text-main); margin-right: 15px; display: flex; justify-content: center; align-items: center; color: white; font-weight: bold; }
        .activity-info { flex: 1; }
        .activity-name { font-size: 16px; font-weight: 700; margin-bottom: 4px; }
        .activity-cue { font-size: 12px; color: var(--text-muted); line-height: 1.4; }

        .log-btn { background: var(--dark-card); color: white; border: none; width: 100%; padding: 16px; border-radius: 15px; font-size: 16px; font-weight: 700; margin-top: 20px; margin-bottom: 40px; cursor: pointer; }
    </style>
</head>
<body>

    <div class="header">
        <p id="workout-count">Total Workouts Logged: 0</p>
        <h1>Ready to work.</h1>
    </div>

    <div class="progress-card">
        <div>
            <div class="progress-title">Protocol Status</div>
            <div class="progress-sub">Stay consistent.</div>
        </div>
        <div class="ring">GO</div>
    </div>

    <div class="toggle-container">
        <div class="toggle-btn" id="btn-gym" onclick="switchPlan('gym')">Gym (Weights)</div>
        <div class="toggle-btn active" id="btn-home" onclick="switchPlan('home')">Home (Floor)</div>
    </div>

    <div class="section-title" id="routine-title">Today's Floor Routine</div>

    <div id="exercise-list">
        </div>

    <button class="log-btn" onclick="logWorkout()">Finish & Log Workout</button>

    <script>
        // THE WORKOUT DATABASE
        const db = {
            gym: [
                { name: 'Barbell Bench Press', load: '5x5', cue: 'Explosive upward drive, 2s controlled descent, full vertical lockout' },
                { name: 'Seated DB Shoulder Press', load: '3x8', cue: 'Lower dumbbells slowly to the temples' },
                { name: 'Barbell Rows (Underhand)', load: '3x10-12', cue: 'Pull to lower stomach, 1s hard contraction squeeze' }
            ],
            home: [
                { name: 'Standard Floor Push-Ups', load: 'Bodyweight', cue: 'Go to technical failure' },
                { name: 'Pike Push-Ups', load: 'Bodyweight', cue: 'Hips jackknifed high. Lower skull towards floor. 3 x Failure' },
                { name: 'Reverse Floor Angels', load: 'Bodyweight', cue: 'Lying prone (face down). Continuous shoulder blade compression.' },
                { name: 'Single-Leg Glute Bridges', load: 'Bodyweight', cue: 'Drive heels into floor. Complete isometric glute lock at peak.' },
                { name: 'Hollow Body Holds', load: 'Bodyweight', cue: 'Press lower lumbar flat into floor. 4 sets to muscular failure' }
            ]
        };

        let currentPlan = 'home';
        let totalWorkouts = localStorage.getItem('creedWorkouts') ? parseInt(localStorage.getItem('creedWorkouts')) : 0;

        // Initialize App
        document.getElementById('workout-count').innerText = `Total Workouts Logged: ${totalWorkouts}`;
        renderExercises();

        function switchPlan(plan) {
            currentPlan = plan;
            
            // Update UI Toggles
            document.getElementById('btn-gym').classList.remove('active');
            document.getElementById('btn-home').classList.remove('active');
            document.getElementById(`btn-${plan}`).classList.add('active');

            // Update Title
            document.getElementById('routine-title').innerText = plan === 'gym' ? "Today's Gym Routine" : "Today's Floor Routine";

            renderExercises();
        }

        function renderExercises() {
            const list = document.getElementById('exercise-list');
            list.innerHTML = ''; 

            db[currentPlan].forEach((ex, index) => {
                const card = document.createElement('div');
                card.className = 'activity-card';
                card.innerHTML = `
                    <div class="activity-icon">${index + 1}</div>
                    <div class="activity-info">
                        <div class="activity-name">${ex.name}</div>
                        <div class="activity-cue">${ex.load} | ${ex.cue}</div>
                    </div>
                `;
                list.appendChild(card);
            });
        }

        function logWorkout() {
            totalWorkouts++;
            localStorage.setItem('creedWorkouts', totalWorkouts);
            document.getElementById('workout-count').innerText = `Total Workouts Logged: ${totalWorkouts}`;
            alert('Protocol executed. Workout saved locally to your device.');
        }
    </script>
</body>
</html>
