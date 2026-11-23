# sherlocked2.github.io

<!DOCTYPE html>
<html lang="cs">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Matrix Cipher</title>

<style>
* {
    box-sizing: border-box;
}

body {
    margin: 0;
    padding: 30px 15px;
    font-family: Arial, sans-serif;
    background: #151923;
    color: #f1f5f9;
}

.container {
    max-width: 900px;
    margin: 30px auto;
    padding: 30px;
    background: #222938;
    border-radius: 14px;
    box-shadow: 0 5px 25px rgba(0,0,0,0.3);
}

h1, h2 {
    color: #61a5ff;
}

h1 {
    text-align: center;
}

.subtitle {
    text-align: center;
    color: #aebbd0;
    margin-bottom: 35px;
}

label {
    display: block;
    margin: 18px 0 8px;
    font-weight: bold;
}

input, select, textarea {
    width: 100%;
    padding: 13px;
    border: 1px solid #46536a;
    border-radius: 7px;
    background: #151923;
    color: white;
    font-size: 16px;
}

input:focus, select:focus, textarea:focus {
    outline: none;
    border-color: #61a5ff;
}

textarea {
    min-height: 110px;
    resize: vertical;
}

.row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 15px;
}

button {
    width: 100%;
    padding: 15px;
    margin-top: 25px;
    background: #3285ed;
    color: white;
    border: none;
    border-radius: 8px;
    font-size: 17px;
    font-weight: bold;
    cursor: pointer;
}

button:hover {
    background: #1c6dcc;
}

.result {
    display: none;
    margin-top: 35px;
    padding: 22px;
    background: #192f40;
    border-radius: 10px;
}

.result textarea {
    background: #111e2b;
    font-size: 18px;
    min-height: 100px;
}

.error {
    display: none;
    margin-top: 20px;
    padding: 15px;
    background: #642d35;
    color: #ffb7b7;
    border-radius: 7px;
    font-weight: bold;
    text-align: center;
}

.dimensions {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 12px;
    margin: 15px 0;
}

.dimension-box {
    padding: 15px;
    background: #263b50;
    border-radius: 8px;
    text-align: center;
}

.dimension-box strong {
    display: block;
    color: #61a5ff;
    font-size: 24px;
    margin-top: 8px;
}

.matrix-section {
    margin-top: 35px;
    overflow-x: auto;
}

.matrix-section h3 {
    color: #61a5ff;
}

table {
    border-collapse: collapse;
    margin: 12px 0;
    width: auto;
    min-width: 100%;
}

td, th {
    border: 1px solid #536176;
    padding: 10px;
    min-width: 35px;
    text-align: center;
    font-size: 15px;
}

th {
    background: #304258;
}

td {
    background: #192433;
}

.empty {
    color: #65758b;
    background: #101720;
}

.small-note {
    color: #9baac0;
    font-size: 13px;
    line-height: 1.6;
}

.copy-button {
    background: #26875a;
    margin-top: 10px;
}

.copy-button:hover {
    background: #1d7049;
}

hr {
    border: none;
    border-top: 1px solid #46536a;
    margin: 30px 0;
}

@media(max-width: 600px) {
    .container {
        padding: 20px;
    }

    .row {
        grid-template-columns: 1fr;
        gap: 0;
    }
}
</style>
</head>

<body>

<div class="container">

<h1>Matrix Cipher</h1>

<p class="subtitle">
Personalized matrix encryption system
</p>

<form id="cipherForm">

<label for="personalNumber">
Personal number (exactly 8 digits)
</label>

<input
    type="text"
    id="personalNumber"
    inputmode="numeric"
    maxlength="8"
    placeholder="Example: 58120447"
    required
>

<label for="gender">
Recipient gender
</label>

<select id="gender" required>
    <option value="FEMALE">Female</option>
    <option value="MALE">Male</option>
    <option value="NONBINARY">Nonbinary</option>
</select>

<label for="cipherKey">
Cipher key
</label>

<input
    type="text"
    id="cipherKey"
    placeholder="Enter cipher key"
    required
>

<label for="message">
Text to encrypt
</label>

<textarea
    id="message"
    placeholder="Enter your message..."
    required
></textarea>

<div class="row">

    <div>
        <label for="rows">Matrix rows</label>
        <input
            type="number"
            id="rows"
            min="1"
            max="50"
            value="5"
            required
        >
    </div>

    <div>
        <label for="cols">Matrix columns</label>
        <input
            type="number"
            id="cols"
            min="1"
            max="50"
            value="3"
            required
        >
    </div>

</div>

<button type="submit">
ENCRYPT MESSAGE
</button>

</form>

<div id="error" class="error"></div>

<div id="result" class="result">

<h2>Encryption result</h2>

<div class="dimensions">

    <div class="dimension-box">
        Actual matrix
        <strong id="actualSize"></strong>
    </div>

    <div class="dimension-box">
        Fibonacci matrix
        <strong id="fibonacciSize"></strong>
    </div>

</div>

<label>Original normalized message</label>
<textarea id="normalizedOutput" readonly></textarea>

<label>After personal shift</label>
<textarea id="personalOutput" readonly></textarea>

<label>Final encrypted message</label>
<textarea id="finalOutput" readonly></textarea>

<button type="button" class="copy-button" onclick="copyResult()">
COPY ENCRYPTED MESSAGE
</button>

<hr>

<h2>Encryption details</h2>

<p class="small-note">
The following matrices show each individual encryption stage.
Empty cells are skipped when reading the result.
</p>

<div id="matrixDetails"></div>

</div>

</div>

<script>

const ALPHABET = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";


// ======================================
// 1. NORMALIZATION
// ======================================

function normalizeText(text) {

    return text
        .normalize("NFD")
        .replace(/[\u0300-\u036f]/g, "")
        .toUpperCase()
        .replace(/[^A-Z]/g, "");

}


// ======================================
// 2. PERSONAL NUMBER SHIFT
// ======================================

function personalShift(text, number, gender) {

    let result = "";

    for (let i = 0; i < text.length; i++) {

        const position = ALPHABET.indexOf(text[i]);

        const digit = Number(number[i % 8]);

        let shift = 0;

        if (gender === "MALE") {

            // Towards A
            shift = -digit;

        } else if (gender === "FEMALE") {

            // Towards Z
            shift = digit;

        } else if (gender === "NONBINARY") {

            // First towards Z, then towards A
            shift = (i % 2 === 0) ? digit : -digit;

        }

        const newPosition =
            (position + shift + 26 * 8) % 26;

        result += ALPHABET[newPosition];

    }

    return result;

}


// ======================================
// 3. FIBONACCI DIMENSIONS
// ======================================

function fibonacciNumber(order) {

    const sequence = [1, 2];

    while (sequence.length < order) {

        const next =
            sequence[sequence.length - 1] +
            sequence[sequence.length - 2];

        sequence.push(next);

    }

    return sequence[order - 1];

}


// ======================================
// 4. GENERATE PRIME NUMBERS
// ======================================

function generatePrimes(count) {

    const primes = [];

    let number = 2;

    while (primes.length < count) {

        let isPrime = true;

        for (let i = 2; i <= Math.sqrt(number); i++) {

            if (number % i === 0) {
                isPrime = false;
                break;
            }

        }

        if (isPrime) {
            primes.push(number);
        }

        number++;

    }

    return primes;

}


// ======================================
// 5. PRIME COLUMN SHIFT
// ======================================

function primeShift(letter, column, primes) {

    if (!letter) return "";

    const position = ALPHABET.indexOf(letter.toUpperCase());

    const shift = primes[column];

    // First column moves towards Z.
    // Second column moves towards A.
    const direction = column % 2 === 0 ? 1 : -1;

    const newPosition =
        (position + direction * shift + 26 * 20) % 26;

    let result = ALPHABET[newPosition];

    // Preserve lowercase letters
    if (letter === letter.toLowerCase()) {
        result = result.toLowerCase();
    }

    return result;

}


// ======================================
// 6. MIDDLE LETTER CALCULATION
// ======================================

function middleLetter(first, second) {

    const a = ALPHABET.indexOf(first);
    const b = ALPHABET.indexOf(second);

    const lower = Math.min(a, b);
    const upper = Math.max(a, b);

    const distance = upper - lower;

    // Identical letters
    if (distance === 0) {
        return ALPHABET[lower];
    }

    const middleDistance = distance / 2;

    // Odd distance: one exact middle letter
    if (distance % 2 === 0) {

        return ALPHABET[lower + middleDistance];

    }

    // Even distance: two middle letters.
    // Choose the earlier one and make it lowercase.

    return ALPHABET[lower + Math.floor(middleDistance)]
        .toLowerCase();

}


// ======================================
// 7. CREATE MATRICES
// ======================================

function createEmptyMatrix(rows, cols) {

    return Array.from(
        { length: rows },
        () => Array(cols).fill("")
    );

}


// Fill message row by row
function fillMessage(matrix, text, start) {

    const rows = matrix.length;
    const cols = matrix[0].length;

    let index = start;

    for (let r = 0; r < rows; r++) {

        for (let c = 0; c < cols; c++) {

            if (index < text.length) {

                matrix[r][c] = text[index];
                index++;

            }

        }

    }

}


// Fill key column by column.
// The key repeats from its beginning when necessary.

function fillKey(matrix, key, startIndex) {

    const rows = matrix.length;
    const cols = matrix[0].length;

    let index = startIndex;

    for (let c = 0; c < cols; c++) {

        for (let r = 0; r < rows; r++) {

            matrix[r][c] = key[index % key.length];

            index++;

        }

    }

    return index;

}


// ======================================
// 8. CREATE HTML MATRIX
// ======================================

function renderMatrix(matrix, title) {

    let html = `
        <h3>${title}</h3>
        <table>
    `;

    for (let r = 0; r < matrix.length; r++) {

        html += "<tr>";

        for (let c = 0; c < matrix[r].length; c++) {

            const value = matrix[r][c];

            if (value === "") {

                html += '<td class="empty">–</td>';

            } else {

                html += `<td>${value}</td>`;

            }

        }

        html += "</tr>";

    }

    html += "</table>";

    return html;

}


// ======================================
// 9. MAIN ENCRYPTION
// ======================================

function encrypt() {

    const personalNumber =
        document.getElementById("personalNumber").value;

    const gender =
        document.getElementById("gender").value;

    const rawKey =
        document.getElementById("cipherKey").value;

    const rawMessage =
        document.getElementById("message").value;

    const rows =
        Number(document.getElementById("rows").value);

    const cols =
        Number(document.getElementById("cols").value);


    // ERROR HANDLING

    if (!/^\d{8}$/.test(personalNumber)) {
        throw new Error("ERROR: Personal number must contain exactly 8 digits.");
    }

    if (!Number.isInteger(rows) || rows < 1 || rows > 50) {
        throw new Error("ERROR: Rows must be between 1 and 50.");
    }

    if (!Number.isInteger(cols) || cols < 1 || cols > 50) {
        throw new Error("ERROR: Columns must be between 1 and 50.");
    }

    if (!rawMessage.trim()) {
        throw new Error("ERROR: Message cannot be empty.");
    }

    if (!rawKey.trim()) {
        throw new Error("ERROR: Cipher key cannot be empty.");
    }


    const message = normalizeText(rawMessage);
    const key = normalizeText(rawKey);

    if (!message.length) {
        throw new Error("ERROR: Message contains no valid letters.");
    }

    if (!key.length) {
        throw new Error("ERROR: Key contains no valid letters.");
    }


    // PERSONAL SHIFT

    const shiftedMessage =
        personalShift(message, personalNumber, gender);


    // FIBONACCI DIMENSIONS

    const fibRows = fibonacciNumber(rows);
    const fibCols = fibonacciNumber(cols);


    // PRIME NUMBERS

    const primes = generatePrimes(cols);


    // MATRIX PROCESSING

    const capacity = rows * cols;

    const numberOfMatrices =
        Math.ceil(shiftedMessage.length / capacity);

    if (numberOfMatrices > 100) {
        throw new Error("ERROR: Too many matrices (maximum 100).");
    }

    let finalResult = "";

    let detailsHTML = "";

    let keyIndex = 0;


    for (let batch = 0; batch < numberOfMatrices; batch++) {

        const start = batch * capacity;

        // Matrix 1: message
        const messageMatrix =
            createEmptyMatrix(rows, cols);

        fillMessage(messageMatrix, shiftedMessage, start);


        // Matrix 2: key
        const keyMatrix =
            createEmptyMatrix(rows, cols);

        keyIndex = fillKey(keyMatrix, key, keyIndex);


        // Matrix 3: middle letters
        const middleMatrix =
            createEmptyMatrix(rows, cols);

        // Matrix 4: final shifted letters
        const finalMatrix =
            createEmptyMatrix(rows, cols);


        for (let r = 0; r < rows; r++) {

            for (let c = 0; c < cols; c++) {

                const messageLetter = messageMatrix[r][c];
                const keyLetter = keyMatrix[r][c];

                // Empty message cells are skipped.
                if (!messageLetter) {
                    continue;
                }

                // Calculate middle letter
                const middle = middleLetter(
                    messageLetter,
                    keyLetter
                );

                middleMatrix[r][c] = middle;


                // Apply prime column shift
                const finalLetter = primeShift(
                    middle,
                    c,
                    primes
                );

                finalMatrix[r][c] = finalLetter;

            }

        }


        // Read final matrix row by row
        for (let r = 0; r < rows; r++) {

            for (let c = 0; c < cols; c++) {

                if (finalMatrix[r][c] !== "") {

                    finalResult += finalMatrix[r][c];

                }

            }

        }


        // DETAILS

        detailsHTML += `
            <div class="matrix-section">

                <h2>Matrix ${batch + 1}</h2>

                ${renderMatrix(
                    messageMatrix,
                    "1. Message matrix (row by row)"
                )}

                ${renderMatrix(
                    keyMatrix,
                    "2. Key matrix (column by column)"
                )}

                ${renderMatrix(
                    middleMatrix,
                    "3. Middle letter matrix"
                )}

                ${renderMatrix(
                    finalMatrix,
                    "4. Final matrix (prime shifts)"
                )}

            </div>
            <hr>
        `;

    }


    // DISPLAY RESULTS

    document.getElementById("actualSize").textContent =
        rows + " × " + cols;

    document.getElementById("fibonacciSize").textContent =
        fibRows + " × " + fibCols;

    document.getElementById("normalizedOutput").value =
        message;

    document.getElementById("personalOutput").value =
        shiftedMessage;

    document.getElementById("finalOutput").value =
        finalResult;

    document.getElementById("matrixDetails").innerHTML =
        detailsHTML;

    document.getElementById("result").style.display =
        "block";

}


// ======================================
// 10. FORM SUBMISSION
// ======================================

document.getElementById("cipherForm")
    .addEventListener("submit", function(event) {

        event.preventDefault();

        const errorBox = document.getElementById("error");

        const resultBox = document.getElementById("result");

        errorBox.style.display = "none";
        resultBox.style.display = "none";

        try {

            encrypt();

        } catch (error) {

            errorBox.textContent = error.message;

            errorBox.style.display = "block";

        }

    });


// ======================================
// 11. COPY RESULT
// ======================================

function copyResult() {

    const text =
        document.getElementById("finalOutput").value;

    navigator.clipboard.writeText(text)
        .then(() => {
            alert("Encrypted message copied!");
        })
        .catch(() => {
            alert("ERROR: Copying failed.");
        });

}

</script>

</body>
</html>
