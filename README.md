<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CGPA Calculator (Local Storage)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            color: #000;
            background-color: #fff;
            transition: background 0.3s, color 0.3s;
        }
        /* Remove @apply and use Tailwind classes in HTML only */
        .input-field {
            width: 100%;
            padding: 0.75rem;
            background-color: #e5e7eb;
            border: 1px solid #9ca3af;
            border-radius: 0.5rem;
            color: #000;
            transition: box-shadow 0.3s;
        }
        .input-field::placeholder {
            color: #6b7280;
        }
        .result-field {
            width: 100%;
            padding: 0.75rem;
            background-color: #f3f4f6;
            border: 1px solid #9ca3af;
            border-radius: 0.5rem;
            color: #15803d;
            font-family: monospace;
            font-size: 1.125rem;
            text-align: center;
        }
        .fade-in {
            animation: fadeIn 0.5s ease-in-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(-10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        body.dark-mode {
            color: #fff;
            background-color: #111827;
        }
        body.dark-mode .input-field {
            background-color: #374151;
            border-color: #4b5563;
            color: #fff;
        }
        body.dark-mode .input-field::placeholder {
            color: #9ca3af;
        }
        body.dark-mode .result-field {
            background-color: #1f2937;
            border-color: #374151;
            color: #4ade80;
        }
    </style>
</head>

<body class="min-h-screen flex items-center justify-center p-4">

    <button id="darkModeToggle" class="absolute top-4 right-4 px-4 py-2 rounded-lg bg-gray-200 text-black dark:bg-gray-700 dark:text-white border border-gray-400 dark:border-gray-600 transition-colors duration-300 z-50">🌙 Dark Mode</button>
    <main id="app-container" class="w-full max-w-4xl bg-gray-100 dark:bg-gray-800/50 backdrop-blur-sm p-6 sm:p-8 rounded-2xl shadow-2xl border border-gray-300 dark:border-gray-700 fade-in">
        <header class="text-center mb-8">
            <h1 class="text-3xl sm:text-4xl font-bold tracking-tight dark:text-white text-black">CGPA Calculator</h1>
            <p class="text-gray-600 dark:text-gray-400 mt-2">Your data is saved locally in this browser.</p>
        </header>

        <!-- Semester Inputs -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-x-8 gap-y-6 mb-8">
            <!-- Semesters will be dynamically inserted here -->
        </div>

        <!-- Results Section -->
        <div class="bg-gray-200 dark:bg-gray-900/60 p-6 rounded-xl border border-gray-300 dark:border-gray-700">
            <h2 class="text-xl font-semibold text-center mb-4 dark:text-white text-black">Your Results</h2>
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                <div>
                    <label for="totalCGPA" class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">Total CGPA</label>
                    <button id="totalCGPA" class="result-field cursor-pointer select-all focus:outline-none" style="width:100%" title="Click to copy CGPA" type="button"></button>
                </div>
                <div>
                    <label for="percentage" class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-1">Percentage</label>
                    <button id="percentage" class="result-field cursor-pointer select-all focus:outline-none" style="width:100%" title="Click to copy Percentage" type="button"></button>
                </div>
            </div>
        </div>
        
        <footer class="text-center mt-8">
             <p class="text-xs text-gray-500 dark:text-gray-400">Storage: Local</p>
        </footer>

    </main>

    <script>
        // --- DARK MODE TOGGLE ---
        const darkModeToggle = document.getElementById('darkModeToggle');
        function setDarkMode(enabled) {
            if (enabled) {
                document.body.classList.add('dark-mode');
                localStorage.setItem('cgpaDarkMode', '1');
                darkModeToggle.textContent = '☀️ Light Mode';
            } else {
                document.body.classList.remove('dark-mode');
                localStorage.setItem('cgpaDarkMode', '0');
                darkModeToggle.textContent = '🌙 Dark Mode';
            }
        }
        darkModeToggle.addEventListener('click', () => {
            setDarkMode(!document.body.classList.contains('dark-mode'));
        });
        // On load, set dark mode if previously enabled
        if (localStorage.getItem('cgpaDarkMode') === '1' || (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
            setDarkMode(true);
        } else {
            setDarkMode(false);
        }
    // --- UI ELEMENTS ---
        const semesterGrid = document.querySelector('.grid.md\\:grid-cols-2');
        const STORAGE_KEY = 'cgpaCalculatorData';

        // --- APPLICATION SETUP ---
        function setupApp() {
            // Generate semester input fields
            for (let i = 1; i <= 8; i++) {
                // Added inputmode="decimal" for CGPA and inputmode="numeric" for Credits
                const semesterHTML = `
                    <div class="grid grid-cols-2 gap-3 items-center">
                        <label class="font-medium text-gray-300">Semester ${i}</label>
                        <div class="grid grid-cols-2 gap-2">
                            <input type="number" inputmode="decimal" id="semester${i}CGPA" placeholder="CGPA" class="input-field semester-input" data-semester="${i}" data-type="cgpa" step="0.01" min="0" max="10" style="color:inherit;background:inherit;">
                            <input type="number" inputmode="numeric" id="semester${i}Credit" placeholder="Credits" class="input-field semester-input" data-semester="${i}" data-type="credit" step="1" min="0" style="color:inherit;background:inherit;">
                        </div>
                    </div>
                `;
                semesterGrid.innerHTML += semesterHTML;
            }

            // Add event listeners to all inputs
            document.querySelectorAll('.semester-input').forEach(input => {
                input.addEventListener('input', handleInputChange);
            });
            
            // Load existing data from local storage
            loadDataFromLocalStorage();
        }

        // --- DATA HANDLING ---

        /**
         * Loads data from localStorage and updates the UI.
         */
        function loadDataFromLocalStorage() {
            const savedData = localStorage.getItem(STORAGE_KEY);
            if (savedData) {
                const data = JSON.parse(savedData);
                console.log("Loaded data from Local Storage:", data);
                updateUI(data.semesters || {});
            } else {
                console.log("No data found in Local Storage. Initializing.");
                calculate(); // Calculate with empty fields initially
            }
        }
        
        /**
         * Saves the current input data to localStorage.
         */
        function saveDataToLocalStorage() {
             const dataToSave = { semesters: {} };
            for (let i = 1; i <= 8; i++) {
                const cgpa = document.getElementById(`semester${i}CGPA`).value;
                const credit = document.getElementById(`semester${i}Credit`).value;
                if (cgpa || credit) { // Only save if there's some data
                    dataToSave.semesters[i] = {
                        cgpa: parseFloat(cgpa) || 0,
                        credit: parseFloat(credit) || 0
                    };
                }
            }
            localStorage.setItem(STORAGE_KEY, JSON.stringify(dataToSave));
            console.log("Data saved to Local Storage.");
        }


        /**
         * Updates the input fields with data.
         * @param {Object} semestersData - The semester data object.
         */
        function updateUI(semestersData) {
            for (let i = 1; i <= 8; i++) {
                const semester = semestersData[i] || {};
                const cgpaInput = document.getElementById(`semester${i}CGPA`);
                const creditInput = document.getElementById(`semester${i}Credit`);
                
                if (cgpaInput) cgpaInput.value = semester.cgpa || '';
                if (creditInput) creditInput.value = semester.credit || '';
            }
            calculate();
        }

        /**
         * Handles input changes, saves data, and recalculates.
         */
        function handleInputChange() {
            saveDataToLocalStorage();
            calculate();
        }

        /**
         * Calculates the total CGPA and percentage based on the current input values.
         */
        function calculate() {
            let totalPoints = 0;
            let totalCredits = 0;

            // Calculate total points and total credits for each semester
            for (let semester = 1; semester <= 8; semester++) {
                const cgpaInput = document.getElementById(`semester${semester}CGPA`);
                const creditInput = document.getElementById(`semester${semester}Credit`);

                if (cgpaInput && creditInput) {
                    const cgpaValue = parseFloat(cgpaInput.value) || 0;
                    const creditValue = parseFloat(creditInput.value) || 0;
                    totalPoints += cgpaValue * creditValue;
                    totalCredits += creditValue;
                }
            }

            const totalCGPAField = document.getElementById("totalCGPA");
            const percentageField = document.getElementById("percentage");

            if (totalCredits === 0) {
                totalCGPAField.textContent = "0.000";
                percentageField.textContent = "0.00%";
                return;
            }

            // Calculate total CGPA and percentage
            const totalCGPA = (totalPoints / totalCredits).toFixed(3);
            const percentage = ((totalCGPA - 0.5) * 10).toFixed(2);

            // Display results
            if (totalCGPAField && percentageField) {
                totalCGPAField.textContent = totalCGPA;
                percentageField.textContent = percentage > 0 ? percentage + "%" : "0.00%";
            } else {
                console.log("Result fields not found");
            }
        // --- COPY TO CLIPBOARD FOR RESULT BUTTONS ---
        function setupCopyButtons() {
            const cgpaBtn = document.getElementById('totalCGPA');
            const percBtn = document.getElementById('percentage');
            function copyText(text, btn, restore) {
                if (navigator.clipboard && window.isSecureContext) {
                    navigator.clipboard.writeText(text).then(() => {
                        btn.innerText = 'Copied!';
                        setTimeout(restore, 1000);
                    });
                } else {
                    // fallback for older browsers
                    const textarea = document.createElement('textarea');
                    textarea.value = text;
                    document.body.appendChild(textarea);
                    textarea.select();
                    try {
                        document.execCommand('copy');
                        btn.innerText = 'Copied!';
                        setTimeout(restore, 1000);
                    } catch (e) {}
                    document.body.removeChild(textarea);
                }
            }
            cgpaBtn.addEventListener('mousedown', function(e) { e.preventDefault(); });
            percBtn.addEventListener('mousedown', function(e) { e.preventDefault(); });
            cgpaBtn.addEventListener('click', function(e) {
                e.preventDefault();
                const val = cgpaBtn.innerText;
                if (val && val !== 'Copied!') {
                    copyText(val, cgpaBtn, () => calculate());
                }
            });
            percBtn.addEventListener('click', function(e) {
                e.preventDefault();
                const val = percBtn.innerText;
                if (val && val !== 'Copied!') {
                    copyText(val, percBtn, () => calculate());
                }
            });
        }
        }

        // --- START THE APP ---
        window.onload = function() {
            setupApp();
            setupCopyButtons();
        };

    </script>
</body>
</html>
