# 1

To improve the update speed and iteration of your web app calculator, you can definitely build and debug it locally using VSCode. Here's a step-by-step guide to get you started:

### Step 1: Set Up VSCode for Local Development

1. **Install VSCode**: Make sure you have [VSCode](https://code.visualstudio.com/) installed on your computer.

2. **Install Live Server Extension**: 
   - Open VSCode.
   - Go to the Extensions view by clicking on the Extensions icon in the Activity Bar on the side of the window or by pressing `Ctrl+Shift+X`.
   - Search for "Live Server" and install it. This extension allows you to launch a local development server with live reload.

### Step 2: Set Up Your Project

1. **Open Your Project**: 
   - Open VSCode.
   - Select `File` > `Open Folder...` and choose the folder containing your project files.

2. **Create Project Structure**: Ensure your project structure looks something like this:
   ```
   your-project-folder/
   ├── assets/
   │   └── js/
   │       └── script.js
   ├── index.html
   └── styles/
       └── style.css
   ```

### Step 3: Run Live Server

1. **Launch Live Server**:
   - Right-click on your `index.html` file in VSCode's Explorer pane.
   - Select `Open with Live Server`. This will start a local server and open your app in the default web browser.
   - Alternatively, you can click the "Go Live" button in the bottom-right corner of VSCode.

### Step 4: Debugging in VSCode

1. **Inspect and Debug**:
   - With Live Server running, you can inspect and debug your code directly in the browser's developer tools (usually opened by pressing `F12` or `Ctrl+Shift+I`).
   - Set breakpoints, step through your code, and see console outputs in real-time.

2. **Modify and Save**:
   - Make changes to your HTML, CSS, or JavaScript files in VSCode.
   - Save the files, and the Live Server will automatically reload the browser to reflect your changes.

### Example: Debugging the "Excellent" Scenario Issue

1. **Check JavaScript for Errors**:
   - Open `script.js` in VSCode.
   - Set breakpoints on lines where you define and use `scenarioVector`, especially in the `selectScenario` function.

2. **Add Console Logs**:
   - Add `console.log` statements to check the values of variables and the flow of execution.
   ```javascript
   function selectScenario(scenario) {
     console.log("Selected scenario: ", scenario); // Log the selected scenario
     switch (scenario) {
       case 'excellent':
         scenarioVector = [1, 0, 0, 0, 0];
         break;
       case 'verygood':
         scenarioVector = [0, 1, 0, 0, 0];
         break;
       case 'good':
         scenarioVector = [0, 0, 1, 0, 0];
         break;
       case 'fair':
         scenarioVector = [0, 0, 0, 1, 0];
         break;
       case 'poor':
         scenarioVector = [0, 0, 0, 0, 1];
         break;
       default:
         scenarioVector = [0, 0, 1, 0, 0]; // Set default to 'good'
     }
     console.log("Scenario vector: ", scenarioVector); // Log the scenario vector
     calculateMortalityRisk(scenario); // Pass selected scenario to calculateMortalityRisk
   }
   ```

3. **Verify Event Listener**:
   - Ensure the event listener is correctly attached and functioning.
   ```javascript
   document.getElementById("scenario-dropdown").addEventListener("change", function() {
     console.log("Dropdown changed"); // Log when the dropdown changes
     selectScenario(this.value);
   });
   ```

By following these steps, you'll be able to develop and debug your web app more efficiently using VSCode. This setup allows for rapid iteration and immediate feedback, significantly speeding up your development process.

# 2

The issue with the "excellent" scenario not displaying a curve might be due to the `scenario` variable not being passed correctly or being used in the `calculateMortalityRisk` function. Here's an adjusted version of the script to ensure that the `scenario` is correctly used for color schemes and data calculation:

```javascript
'use strict';

var scenarioVector = [1, 0, 0, 0, 0]; // Default to excellent self-rated health scenario

function selectScenario(scenario) {
  switch (scenario) {
    case 'excellent':
      scenarioVector = [1, 0, 0, 0, 0];
      break;
    case 'verygood':
      scenarioVector = [0, 1, 0, 0, 0];
      break;
    case 'good':
      scenarioVector = [0, 0, 1, 0, 0];
      break;
    case 'fair':
      scenarioVector = [0, 0, 0, 1, 0];
      break;
    case 'poor':
      scenarioVector = [0, 0, 0, 0, 1];
      break;
    default:
      scenarioVector = [0, 0, 1, 0, 0]; // Set default to 'good'
  }
  calculateMortalityRisk(scenario); // Pass selected scenario to calculateMortalityRisk
}

function calculateMortalityRisk(scenario) {
  const beta = [0, .29266961, .63127178, 1.0919233, 2.010895]; // Beta coefficients for excellent, very good, good, fair, poor
  const s0 = [.9999999, .96281503, .91558171, .87179276, .82403985]; // Survival probabilities at timepoints 0, 5, 10, 15, 20
  const timePoints = [0, 5, 10, 15, 20];
  const logHR = beta.reduce((acc, curr, index) => acc + (curr * scenarioVector[index]), 0);
  const f0 = s0.map(s => (1 - s) * 100);
  const f1 = f0.map((f, index) => f * Math.exp(logHR));

  // Color schemes for different scenarios
  const colorSchemes = {
    'excellent': 'rgba(0, 191, 255, 1)',    // Bright blue
    'verygood': 'rgba(255, 0, 255, 1)',      // Magenta
    'good': 'rgba(106, 168, 79, 1)',         // Cabbage green
    'fair': 'rgba(255, 255, 0, 1)',          // Lemon yellow
    'poor': 'rgba(128, 0, 128, 1)'           // Purple
  };

  const riskResults = timePoints.map((time, index) => `Risk at ${time} years: ${f1[index].toFixed(2)}%`);

  // Clear existing chart before creating a new one
  if (window.mortalityChart) {
    window.mortalityChart.destroy();
  }

  // Draw graph
  const ctx = document.getElementById('mortality-risk-graph').getContext('2d');
  window.mortalityChart = new Chart(ctx, {
    type: 'line',
    data: {
      labels: timePoints.map(String),
      datasets: [{
        label: 'Mortality Risk',
        data: f1,
        steppedLine: 'before',
        borderColor: colorSchemes[scenario],
        backgroundColor: colorSchemes[scenario].replace('1)', '0.2)'),
        borderWidth: 3
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      scales: {
        x: {
          title: {
            display: true,
            text: 'Timepoints (years)'
          }
        },
        y: {
          title: {
            display: true,
            text: 'Mortality Risk (%)'
          },
          suggestedMin: 0,
          suggestedMax: 80,
          stepSize: 20,
          fontSize: 14
        }
      }
    }
  });

  document.getElementById("mortality-risk-results").innerText = riskResults.join('\n');
}

// Attach event listener to the dropdown to update the scenarioVector
document.getElementById("scenario-dropdown").addEventListener("change", function() {
  selectScenario(this.value);
});
```

**Key changes:**

1. Added `scenario` as an argument to `calculateMortalityRisk` to ensure the correct color scheme is used.
2. Added a check to destroy any existing chart before creating a new one to avoid overlapping charts.

Ensure the rest of your HTML and JavaScript setup remains the same, and this should address the issue with the "excellent" scenario curve not displaying.
