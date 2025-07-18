<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Employee Budgeting and Layoff Planner</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/moment.js/2.29.4/moment.min.js"></script>
    <script src="https://unpkg.com/javascript-lp-solver/prod/solver.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.25/jspdf.plugin.autotable.min.js"></script>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        input, button, textarea { margin: 10px 0; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f2f2f2; }
        #status { color: green; }
        #warning { color: red; }
    </style>
</head>
<body>
    <h1>Employee Budgeting and Layoff Planner</h1>

    <label for="employeeFile">Upload Employee Data (Excel: LastName, FirstName, Position, HourlyPay)</label><br>
    <input type="file" id="employeeFile" accept=".xlsx"><br>

    <label for="scheduleFile">Upload Work Schedule (Excel: Date (YYYY-MM-DD), Hours)</label><br>
    <input type="file" id="scheduleFile" accept=".xlsx"><br>

    <label for="newHires">Add New Hires (one per line: LastName,FirstName,Position,HourlyPay,StartDate (YYYY-MM-DD))</label><br>
    <textarea id="newHires" rows="5" cols="50"></textarea><br>

    <label for="budget">Total Budget</label><br>
    <input type="number" id="budget" step="0.01" min="0"><br>

    <button onclick="loadDataAndCompute()">Compute Optimal Plan</button><br>

    <h2>Proposed Plan</h2>
    <table id="planTable">
        <thead>
            <tr>
                <th>LastName</th>
                <th>FirstName</th>
                <th>Position</th>
                <th>HourlyPay</th>
                <th>StartDate</th>
                <th>ProposedLayoff</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>

    <button onclick="recalculateCost()" style="display:none;" id="recalcBtn">Recalculate Cost with Edits</button>
    <p id="status"></p>
    <p id="warning"></p>

    <button onclick="generatePDF()" style="display:none;" id="pdfBtn">Generate and Download PDF</button>

    <script>
        let employees = [];
        let schedule = [];
        let workDates = [];
        let dailyHours = {};
        let dailyMultipliers = {};
        let possibleEnds = [];
        let noLayoffDate = null;
        let budget = 0;
        let currentTotalCost = 0;

        function parseExcel(file, callback) {
            const reader = new FileReader();
            reader.onload = function(e) {
                const data = e.target.result;
                const workbook = XLSX.read(data, { type: 'binary', cellDates: true });
                const sheetName = workbook.SheetNames[0];
                const sheet = workbook.Sheets[sheetName];
                const json = XLSX.utils.sheet_to_json(sheet, { dateNF: 'yyyy-mm-dd' });
                callback(json);
            };
            reader.readAsBinaryString(file);
        }

        function loadDataAndCompute() {
            const employeeFile = document.getElementById('employeeFile').files[0];
            const scheduleFile = document.getElementById('scheduleFile').files[0];
            budget = parseFloat(document.getElementById('budget').value);

            if (!employeeFile || !scheduleFile || isNaN(budget)) {
                alert('Please upload both files and enter a budget.');
                return;
            }

            parseExcel(scheduleFile, function(sched) {
                schedule = sched;
                workDates = schedule.map(row => moment(row.Date, 'YYYY-MM-DD'));
                workDates.sort((a,b) => a - b);
                const startDate = workDates[0];
                const endDate = workDates[workDates.length - 1];
                noLayoffDate = endDate.clone().add(1, 'days');

                dailyHours = {};
                schedule.forEach(row => {
                    const d = moment(row.Date, 'YYYY-MM-DD');
                    dailyHours[d.format('YYYY-MM-DD')] = row.Hours;
                });

                dailyMultipliers = {};
                workDates.forEach(d => {
                    const wd = d.day(); // 0=Sun, 6=Sat
                    let mult = 1.0;
                    if (wd === 6) mult = 1.5; // Saturday
                    if (wd === 0) mult = 2.0; // Sunday
                    dailyMultipliers[d.format('YYYY-MM-DD')] = mult;
                });

                // Possible layoff dates: Fridays (weekday 5 in moment, 0=Sun,5=Fri,6=Sat)
                let fridayDates = workDates.filter(d => d.day() === 5).sort((a,b) => a - b);
                fridayDates = [...new Set(fridayDates.map(d => d.format('YYYY-MM-DD')))].map(d => moment(d));
                possibleEnds = fridayDates.concat([noLayoffDate]);

                parseExcel(employeeFile, function(emps) {
                    employees = emps.map(emp => ({
                        ...emp,
                        HourlyPay: parseFloat(emp.HourlyPay),
                        StartDate: emp.StartDate ? moment(emp.StartDate, 'YYYY-MM-DD') : startDate.clone()
                    }));

                    // Add new hires
                    const newHiresText = document.getElementById('newHires').value;
                    if (newHiresText) {
                        newHiresText.split('\n').forEach(line => {
                            if (line.trim()) {
                                const parts = line.split(',');
                                if (parts.length === 5) {
                                    const [last, first, pos, pay, hDate] = parts.map(p => p.trim());
                                    employees.push({
                                        LastName: last,
                                        FirstName: first,
                                        Position: pos,
                                        HourlyPay: parseFloat(pay),
                                        StartDate: moment(hDate, 'YYYY-MM-DD')
                                    });
                                }
                            }
                        });
                    }

                    computeOptimalPlan();
                });
            });
        }

        function precomputeMatrices() {
            const numEmps = employees.length;
            const numEnds = possibleEnds.length;
            const costMatrix = Array.from({length: numEmps}, () => Array(numEnds).fill(0));
            const hoursMatrix = Array.from({length: numEmps}, () => Array(numEnds).fill(0));

            for (let i = 0; i < numEmps; i++) {
                const startI = employees[i].StartDate;
                const payI = employees[i].HourlyPay;
                for (let j = 0; j < numEnds; j++) {
                    const endJ = possibleEnds[j];
                    let totalCost = 0;
                    let totalHours = 0;
                    workDates.forEach(d => {
                        if (d.isSameOrAfter(startI) && d.isBefore(endJ)) {
                            const h = dailyHours[d.format('YYYY-MM-DD')];
                            const mult = dailyMultipliers[d.format('YYYY-MM-DD')];
                            totalCost += h * payI * mult;
                            totalHours += h;
                        }
                    });
                    costMatrix[i][j] = totalCost;
                    hoursMatrix[i][j] = totalHours;
                }
            }
            return { costMatrix, hoursMatrix };
        }

        function computeOptimalPlan() {
            const { costMatrix, hoursMatrix } = precomputeMatrices();

            // Set up LP model
            const model = {
                optimize: "totalHours",
                opType: "max",
                constraints: { budget: { max: budget } },
                variables: {},
                ints: {}
            };

            for (let i = 0; i < employees.length; i++) {
                const oneConstraint = { min: 1, max: 1 };
                model.constraints[`emp_${i}`] = oneConstraint;
                for (let j = 0; j < possibleEnds.length; j++) {
                    const varName = `x_${i}_${j}`;
                    model.variables[varName] = {
                        totalHours: hoursMatrix[i][j],
                        budget: costMatrix[i][j],
                        [`emp_${i}`]: 1
                    };
                    model.ints[varName] = 1;
                }
            }

            const results = solver.Solve(model);

            if (results.feasible) {
                const layoffPlan = [];
                for (let i = 0; i < employees.length; i++) {
                    for (let j = 0; j < possibleEnds.length; j++) {
                        const varName = `x_${i}_${j}`;
                        if (results[varName] === 1) {
                            const endJ = possibleEnds[j];
                            const layoff = endJ.isSame(noLayoffDate) ? 'None (full period)' : endJ.format('YYYY-MM-DD');
                            layoffPlan.push(layoff);
                            break;
                        }
                    }
                }

                employees.forEach((emp, idx) => {
                    emp.ProposedLayoff = layoffPlan[idx];
                });

                currentTotalCost = results.budget; // Actually the used cost

                displayPlanTable();
                document.getElementById('recalcBtn').style.display = 'block';
                document.getElementById('pdfBtn').style.display = 'block';
                updateCostDisplay();
            } else {
                alert('No feasible solution found. Try increasing the budget.');
            }
        }

        function displayPlanTable() {
            const tbody = document.getElementById('planTable').querySelector('tbody');
            tbody.innerHTML = '';
            employees.forEach((emp, idx) => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${emp.LastName}</td>
                    <td>${emp.FirstName}</td>
                    <td>${emp.Position}</td>
                    <td>${emp.HourlyPay.toFixed(2)}</td>
                    <td>${emp.StartDate.format('YYYY-MM-DD')}</td>
                    <td contenteditable="true" id="layoff_${idx}">${emp.ProposedLayoff}</td>
                `;
                tbody.appendChild(row);
            });
        }

        function recalculateCost() {
            let totalCost = 0;
            employees.forEach((emp, idx) => {
                let layoff = document.getElementById(`layoff_${idx}`).innerText.trim();
                let endI;
                if (layoff === 'None (full period)') {
                    endI = noLayoffDate;
                } else {
                    endI = moment(layoff, 'YYYY-MM-DD').add(1, 'days'); // Work up to layoff day
                    if (!endI.isValid()) {
                        alert(`Invalid date for employee ${idx}: ${layoff}`);
                        return;
                    }
                }
                const startI = emp.StartDate;
                const payI = emp.HourlyPay;
                workDates.forEach(d => {
                    if (d.isSameOrAfter(startI) && d.isBefore(endI)) {
                        const h = dailyHours[d.format('YYYY-MM-DD')];
                        const mult = dailyMultipliers[d.format('YYYY-MM-DD')];
                        totalCost += h * payI * mult;
                    }
                });
            });
            currentTotalCost = totalCost;
            updateCostDisplay();
        }

        function updateCostDisplay() {
            document.getElementById('status').innerText = `Total calculated cost: $${currentTotalCost.toFixed(2)}`;
            if (currentTotalCost > budget) {
                document.getElementById('warning').innerText = 'Warning: Over budget!';
            } else {
                document.getElementById('warning').innerText = '';
            }
        }

        function generatePDF() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();

            doc.text('Layoff and Work Plan', 10, 10);
            doc.text(`Budget: $${budget.toFixed(2)}`, 10, 20);
            doc.text(`Total Cost: $${currentTotalCost.toFixed(2)}`, 10, 30);

            const tableData = employees.map((emp, idx) => [
                emp.LastName,
                emp.FirstName,
                emp.Position,
                emp.HourlyPay.toFixed(2),
                emp.StartDate.format('YYYY-MM-DD'),
                document.getElementById(`layoff_${idx}`).innerText
            ]);

            doc.autoTable({
                head: [['LastName', 'FirstName', 'Position', 'HourlyPay', 'StartDate', 'ProposedLayoff']],
                body: tableData,
                startY: 40
            });

            doc.save('layoff_plan.pdf');
        }
    </script>
</body>
</html>
