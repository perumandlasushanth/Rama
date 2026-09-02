<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Subject Checklist</title>

<style>
* {
    box-sizing: border-box;
}

body {
    margin: 0;
    padding: 20px 14px;
    background: #f4f6f8;
    font-family: Arial, sans-serif;
}

.card {
    max-width: 560px;
    margin: auto;
    background: white;
    border-radius: 18px;
    padding: 22px;
    box-shadow: 0 4px 18px rgba(0,0,0,0.1);
}

h1 {
    text-align: center;
    margin: 0 0 8px;
}

#progress {
    text-align: center;
    color: #555;
    margin-bottom: 18px;
    font-size: 16px;
}

.item {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 12px 6px;
    border-bottom: 1px solid #eee;
    font-size: 18px;
}

.sub {
    padding-left: 48px;
    font-size: 16px;
}

input[type="checkbox"] {
    width: 23px;
    height: 23px;
    accent-color: #2563eb;
    flex: none;
}

.done span {
    text-decoration: line-through;
    color: #888;
}

#remaining {
    margin-bottom: 15px;
}

.remaining-box {
    padding: 12px;
    border-radius: 10px;
    background: #f0f4ff;
}

.completed-box {
    padding: 12px;
    border-radius: 10px;
    background: #eaf8ee;
}

textarea {
    width: 100%;
    min-height: 110px;
    padding: 12px;
    border: 1px solid #ccc;
    border-radius: 10px;
    font-size: 16px;
    resize: vertical;
}

.notes-title {
    display: block;
    font-weight: bold;
    margin-bottom: 8px;
}

button {
    width: 100%;
    margin-top: 20px;
    padding: 13px;
    border: 0;
    border-radius: 10px;
    background: #2563eb;
    color: white;
    font-size: 16px;
    cursor: pointer;
}

button:active {
    transform: scale(0.98);
}

.toggle-row {
    display: flex;
    gap: 10px;
    margin-top: 10px;
}

.toggle-row button {
    margin-top: 0;
    background: #e5e9f2;
    color: #2563eb;
    font-weight: bold;
}

.toggle-row button.active {
    background: #2563eb;
    color: white;
}

.preview-box {
    display: none;
    padding: 14px;
    border-radius: 10px;
    background: #f9fafb;
    border: 1px solid #e2e5ea;
    min-height: 110px;
    white-space: pre-wrap;
    font-size: 16px;
    color: #222;
}

.preview-box.empty {
    color: #999;
    font-style: italic;
}

.export-btn {
    background: #16a34a;
}
</style>
</head>

<body>

<div class="card">

    <h1>📚 Subject Checklist</h1>

    <div id="progress"></div>

    <!-- Checklist -->
    <div id="list"></div>

    <!-- Update / Notes -->
    <div style="margin-top:20px;">
        <label for="notes" class="notes-title">
            📝 Update / Notes
        </label>

        <textarea
            id="notes"
            placeholder="Write your update here..."
        ></textarea>
    </div>

    <button onclick="resetChecklist()">
        Reset Checklist
    </button>

    <!-- Final Remarks (written last, same save/load feature as Notes) -->
    <div style="margin-top:20px;">
        <label for="finalRemarks" class="notes-title">
            ✅ Final Remarks
        </label>

        <div class="toggle-row">
            <button type="button" id="editBtn" onclick="showEditMode()">
                ✏️ Edit
            </button>
            <button type="button" id="previewBtn" onclick="showPreviewMode()">
                👁️ Preview Result
            </button>
        </div>

        <textarea
            id="finalRemarks"
            placeholder="Write your final remarks here..."
            style="margin-top:12px;"
        ></textarea>

        <div id="finalRemarksPreview" class="preview-box" style="margin-top:12px;"></div>
    </div>

    <button class="export-btn" onclick="exportChecklist()">
        ⬇️ Download Progress as Text File
    </button>

    <!-- Remaining items (shown last) -->
    <div id="remaining" style="margin-top:20px;"></div>

</div>

<script>

/* =========================
   SUBJECTS AND SUB-TOPICS
   ========================= */

const data = [

    ["BEE", [
        "class notes"
    ]],

    ["M&C", []],

    ["EG-CAD", [
        "observation",
        "sheet"
    ]],

    ["PPS", [
        "class notes"
    ]],

    ["ITWS", [
        "lab notes",
        "record"
    ]],

    ["EC", [
        "class notes"
    ]],

    ["EC LAB", [
        "manual",
        "record"
    ]],

    ["BEE LAB", [
        "lab notes",
        "record"
    ]],

    ["PPS LAB", [
        "Lab notes"
    ]]
];


/* =========================
   CREATE CHECKLIST
   ========================= */

const list = document.getElementById("list");

data.forEach((subject, i) => {

    // Main subject
    addItem(
        `${i + 1}. ${subject[0]}`,
        `main-${i}`,
        "main"
    );

    // Sub-topics
    subject[1].forEach((sub, j) => {

        addItem(
            `${String.fromCharCode(105 + j)}) ${sub}`,
            `sub-${i}-${j}`,
            "sub"
        );

    });

});


/* =========================
   ADD ITEM
   ========================= */

function addItem(text, id, type) {

    const label = document.createElement("label");

    label.className = "item " + type;

    const box = document.createElement("input");

    const span = document.createElement("span");

    box.type = "checkbox";

    box.id = id;

    // Load saved status
    box.checked =
        localStorage.getItem(id) === "true";

    span.textContent = text;

    label.appendChild(box);
    label.appendChild(span);

    list.appendChild(label);


    // Show crossed-out text if already completed
    label.classList.toggle(
        "done",
        box.checked
    );


    /* =========================
       CHECKBOX EVENT
       ========================= */

    box.addEventListener("change", () => {

        localStorage.setItem(
            id,
            box.checked
        );

        label.classList.toggle(
            "done",
            box.checked
        );


        /*
         If MAIN SUBJECT is ticked,
         tick all its sub-topics.
        */

        if (type === "main") {

            const subjectNumber =
                id.replace("main-", "");

            const prefix =
                "sub-" + subjectNumber + "-";

            document
                .querySelectorAll(
                    `input[id^="${prefix}"]`
                )
                .forEach(subBox => {

                    subBox.checked =
                        box.checked;

                    localStorage.setItem(
                        subBox.id,
                        subBox.checked
                    );

                    subBox
                        .closest("label")
                        .classList.toggle(
                            "done",
                            subBox.checked
                        );

                });

        }

        /*
         Update main subject automatically
         if all its sub-topics are completed.
        */

        if (type === "sub") {

            const parts = id.split("-");

            const subjectNumber = parts[1];

            const mainBox =
                document.getElementById(
                    `main-${subjectNumber}`
                );

            const subBoxes =
                document.querySelectorAll(
                    `input[id^="sub-${subjectNumber}-"]`
                );

            const allDone =
                [...subBoxes].every(
                    sub => sub.checked
                );

            if (subBoxes.length > 0 && allDone) {

                mainBox.checked = true;

                localStorage.setItem(
                    mainBox.id,
                    true
                );

                mainBox
                    .closest("label")
                    .classList.add("done");

            } else {

                mainBox.checked = false;

                localStorage.setItem(
                    mainBox.id,
                    false
                );

                mainBox
                    .closest("label")
                    .classList.remove("done");

            }

        }

        update();

        // Keep the preview in sync if it's currently visible
        if (
            typeof finalRemarksPreview !== "undefined" &&
            finalRemarksPreview.style.display === "block"
        ) {
            renderFinalRemarksPreview();
        }

    });

}


/* =========================
   UPDATE PROGRESS
   ========================= */

function update() {

    const boxes =
        document.querySelectorAll(
            'input[type="checkbox"]'
        );

    const done =
        [...boxes].filter(
            box => box.checked
        ).length;

    document.getElementById(
        "progress"
    ).textContent =
        `${done} / ${boxes.length} completed`;


    /* =========================
       SHOW REMAINING
       ========================= */

    const remaining =
        [...boxes].filter(
            box => !box.checked
        );

    const remainingDiv =
        document.getElementById(
            "remaining"
        );


    if (remaining.length === 0) {

        remainingDiv.innerHTML = `
            <div class="completed-box">
                <strong>
                    🎉 Everything is completed!
                </strong>
            </div>
        `;

    } else {

        const names =
            remaining.map(box => {

                const text =
                    box
                    .closest("label")
                    .querySelector("span")
                    .textContent;

                return `<li>${text}</li>`;

            }).join("");


        remainingDiv.innerHTML = `
            <div class="remaining-box">

                <strong>
                    📌 Remaining to complete
                    (${remaining.length}):
                </strong>

                <ul>
                    ${names}
                </ul>

            </div>
        `;

    }

}


/* =========================
   NOTES / UPDATE BOX
   ========================= */

const notes =
    document.getElementById("notes");


/* Load saved notes */

notes.value =
    localStorage.getItem(
        "checklistNotes"
    ) || "";


/* Save notes automatically */

notes.addEventListener(
    "input",
    () => {

        localStorage.setItem(
            "checklistNotes",
            notes.value
        );

    }
);


/* =========================
   FINAL REMARKS BOX
   ========================= */

const finalRemarks =
    document.getElementById("finalRemarks");

const finalRemarksPreview =
    document.getElementById("finalRemarksPreview");

const editBtn =
    document.getElementById("editBtn");

const previewBtn =
    document.getElementById("previewBtn");


/* Load saved final remarks */

finalRemarks.value =
    localStorage.getItem(
        "checklistFinalRemarks"
    ) || "";


/* Save final remarks automatically */

finalRemarks.addEventListener(
    "input",
    () => {

        localStorage.setItem(
            "checklistFinalRemarks",
            finalRemarks.value
        );

    }
);


/* Render the preview text */

function renderFinalRemarksPreview() {

    const text = finalRemarks.value.trim();

    const boxes =
        document.querySelectorAll(
            'input[type="checkbox"]'
        );

    const remainingBoxes =
        [...boxes].filter(
            box => !box.checked
        );

    let html = "";

    // Final remarks text part
    if (text === "") {

        html += `<div class="empty">Nothing written yet.</div>`;

    } else {

        html += `<div>${text}</div>`;

    }

    // Remaining (not ticked) items part
    html += `<hr style="margin:14px 0;border:none;border-top:1px solid #e2e5ea;">`;

    if (remainingBoxes.length === 0) {

        html += `<strong>🎉 Everything is ticked — nothing remaining!</strong>`;

    } else {

        const names =
            remainingBoxes.map(box => {

                const itemText =
                    box
                    .closest("label")
                    .querySelector("span")
                    .textContent;

                return `<li>${itemText}</li>`;

            }).join("");

        html += `
            <strong>📌 Still not ticked (${remainingBoxes.length}):</strong>
            <ul style="margin:8px 0 0;padding-left:20px;">
                ${names}
            </ul>
        `;

    }

    finalRemarksPreview.innerHTML = html;

    finalRemarksPreview.classList.toggle(
        "empty",
        false
    );

}


/* Switch to Edit mode */

function showEditMode() {

    finalRemarks.style.display = "block";
    finalRemarksPreview.style.display = "none";

    editBtn.classList.add("active");
    previewBtn.classList.remove("active");

}


/* Switch to Preview mode */

function showPreviewMode() {

    renderFinalRemarksPreview();

    finalRemarks.style.display = "none";
    finalRemarksPreview.style.display = "block";

    previewBtn.classList.add("active");
    editBtn.classList.remove("active");

}


/* Start in Edit mode */

showEditMode();


/* =========================
   RESET CHECKLIST
   ========================= */

function resetChecklist() {

    const boxes =
        document.querySelectorAll(
            'input[type="checkbox"]'
        );


    boxes.forEach(box => {

        box.checked = false;

        localStorage.setItem(
            box.id,
            false
        );

        box
            .closest("label")
            .classList.remove("done");

    });


    update();

}


/* =========================
   EXPORT / DOWNLOAD
   ========================= */

function exportChecklist() {

    let lines = [];

    lines.push("SUBJECT CHECKLIST — PROGRESS EXPORT");
    lines.push("Generated: " + new Date().toLocaleString());
    lines.push("");

    data.forEach((subject, i) => {

        const mainBox =
            document.getElementById(`main-${i}`);

        const mark = mainBox.checked ? "[x]" : "[ ]";

        lines.push(`${mark} ${i + 1}. ${subject[0]}`);

        subject[1].forEach((sub, j) => {

            const subBox =
                document.getElementById(`sub-${i}-${j}`);

            const subMark = subBox.checked ? "[x]" : "[ ]";

            lines.push(`    ${subMark} ${String.fromCharCode(105 + j)}) ${sub}`);

        });

    });

    lines.push("");

    const boxes =
        document.querySelectorAll('input[type="checkbox"]');

    const doneCount =
        [...boxes].filter(box => box.checked).length;

    lines.push(`Progress: ${doneCount} / ${boxes.length} completed`);
    lines.push("");

    lines.push("--- Update / Notes ---");
    lines.push(notes.value.trim() || "(none)");
    lines.push("");

    lines.push("--- Final Remarks ---");
    lines.push(finalRemarks.value.trim() || "(none)");

    const content = lines.join("\n");

    const blob = new Blob([content], { type: "text/plain" });

    const url = URL.createObjectURL(blob);

    const a = document.createElement("a");

    a.href = url;

    a.download = "subject-checklist-progress.txt";

    document.body.appendChild(a);

    a.click();

    document.body.removeChild(a);

    URL.revokeObjectURL(url);

}


/* =========================
   INITIAL UPDATE
   ========================= */

update();

</script>

</body>
</html>
