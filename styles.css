// Funktion zum Abrufen eines Abfrageparameters nach Name
function getQueryParam(param) {
    const urlParams = new URLSearchParams(window.location.search);
    return urlParams.get(param);
}

// Funktion zum Extrahieren des Seitentitels aus der Referrer-URL
function getParentPageTitle() {
    const referrer = document.referrer;
    if (!referrer) {
        console.warn('Kein Referrer gefunden. Der übergeordnete Seitentitel kann nicht abgerufen werden.');
        return '';
    }

    try {
        const url = new URL(referrer);
        const pathSegments = url.pathname.split('/').filter(segment => segment.length > 0);
        const targetSegment = 'allgemeinbildung';
        const targetIndex = pathSegments.indexOf(targetSegment);

        if (targetIndex === -1) {
            console.warn(`Segment '${targetSegment}' wurde im Pfad der Referrer-URL nicht gefunden.`);
            return '';
        }

        const relevantSegments = pathSegments.slice(targetIndex + 1);
        if (relevantSegments.length === 0) {
            console.warn('Keine Pfadsegmente nach dem Zielsegment gefunden.');
            return '';
        }

        const formattedSegments = relevantSegments.map(segment => {
            return decodeURIComponent(segment.replace(/[-_+]/g, ' ')).replace(/\b\w/g, char => char.toUpperCase());
        });
        return formattedSegments.join(' - ');
    } catch (e) {
        console.error('Fehler beim Parsen der Referrer-URL:', e);
        return '';
    }
}

const STORAGE_PREFIX = 'boxsuk-assignment_';
const assignmentId = getQueryParam('assignmentId') || 'defaultAssignment';
const parentTitle = getParentPageTitle();
const assignmentSuffix = assignmentId.replace(/^assignment[_-]?/i, '');

// Setze die Assignment-ID im Titel
const assignmentInfoFragen = document.getElementById('fragenAssignmentId');
if (assignmentInfoFragen) {
    assignmentInfoFragen.textContent = assignmentSuffix || 'defaultAssignment';
}

// Initialisiere den Quill-Editor für Fragen
let quillFragen;
if (document.getElementById('fragenBox')) {
    quillFragen = new Quill('#fragenBox', {
        theme: 'snow',
        placeholder: 'Gib hier deine Antwort ein...',
        modules: {
            toolbar: [
                ['bold', 'italic', 'underline'],
                [{ 'list': 'ordered' }, { 'list': 'bullet' }],
                ['clean']
            ]
        }
    });

    // Setze initialen Inhalt mit einer nummerierten Liste
    quillFragen.clipboard.dangerouslyPasteHTML('<ol><li></li></ol>');

    // Blockiere das Einfügen von Inhalten
    quillFragen.root.addEventListener('paste', function(e) {
        e.preventDefault();
        alert("Einfügen von Inhalten ist in diesem Editor deaktiviert.");
    });
}

// Anzeigeelemente für gespeicherte Antworten (nur Fragen)
const savedAnswerContainer = document.getElementById('savedAnswerContainer');
const savedFragenTitle = document.getElementById('savedFragenTitle');
const savedFragenAnswer = document.getElementById('savedFragenAnswer');
const saveIndicator = document.getElementById('saveIndicator');

// Funktion zum Kopieren von Text in die Zwischenablage
function copyTextToClipboard(text) {
    if (navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard.writeText(text).then(function() {
            console.log("Text erfolgreich kopiert");
        }, function(err) {
            console.error('Fehler beim Kopieren des Textes: ', err);
            fallbackCopyTextToClipboard(text);
        });
    } else {
        fallbackCopyTextToClipboard(text);
    }
}

function fallbackCopyTextToClipboard(text) {
    const textarea = document.createElement("textarea");
    textarea.value = text;
    textarea.style.position = "fixed";
    textarea.style.top = "-9999px";
    document.body.appendChild(textarea);
    textarea.focus();
    textarea.select();

    try {
        const successful = document.execCommand('copy');
        if (successful) {
            console.log("Text erfolgreich kopiert (Fallback)");
        } else {
            throw new Error("Fallback-Kopieren nicht erfolgreich");
        }
    } catch (err) {
        console.error('Fehler beim Kopieren des Textes (Fallback): ', err);
    }
    document.body.removeChild(textarea);
}

// Funktion zum Speichern des Textes in localStorage (nur Fragen)
function saveToLocal() {
    if (!quillFragen) return;
    const fragenContent = quillFragen.root.innerHTML;
    const fragenText = quillFragen.getText().trim();

    if (fragenText === "") {
        console.log("Versuch, mit leerem Textfeld zu speichern");
        return;
    }

    const storageKey = STORAGE_PREFIX + assignmentId;
    const contentToSave = { fragen: fragenContent };
    localStorage.setItem(storageKey, JSON.stringify(contentToSave));
    console.log(`Text für ${storageKey} gespeichert`);
    showSaveIndicator();
}

function showSaveIndicator() {
    const saveIndicator = document.getElementById('saveIndicator');
    saveIndicator.style.backgroundColor = '#4CAF50';
    setTimeout(() => {
        saveIndicator.style.backgroundColor = '#555555';
    }, 1000);
}

// Funktion zum Löschen aller gespeicherten Texte aus localStorage
function clearLocalStorage() {
    const keysToRemove = [];
    for (let i = 0; i < localStorage.length; i++) {
        const key = localStorage.key(i);
        if (key.startsWith(STORAGE_PREFIX)) {
            keysToRemove.push(key);
        }
    }
    keysToRemove.forEach(key => localStorage.removeItem(key));
    if (quillFragen) quillFragen.setText('');
    if (savedAnswerContainer) savedAnswerContainer.style.display = 'none';
    console.log("Alle gespeicherten boxsuk-Texte wurden gelöscht");
    showSaveIndicator();
    loadAllAnswers();
}

// Funktion zum Löschen ausgewählter Antworten (Bulk)
function bulkDeleteAnswers() {
    const selectedCheckboxes = document.querySelectorAll(".select-answer:checked");
    if (selectedCheckboxes.length === 0) {
        alert("Bitte wählen Sie mindestens eine Antwort zum Löschen aus.");
        return;
    }
    if (!confirm(`Sind Sie sicher, dass Sie ${selectedCheckboxes.length} ausgewählte Antwort(en) löschen möchten?`)) {
        return;
    }
    selectedCheckboxes.forEach(cb => {
        const assignmentId = cb.value;
        localStorage.removeItem(assignmentId);
        console.log(`Antwort für ${assignmentId} gelöscht.`);
    });
    alert(`${selectedCheckboxes.length} Antwort(en) wurden gelöscht.`);
    loadAllAnswers();
}

// Funktion zum Drucken einer einzelnen Antwort
function printSingleAnswer(title, content) {
    const printDiv = document.createElement('div');
    printDiv.id = 'printSingleContent';

    const titleElement = document.createElement('h2');
    titleElement.textContent = title;
    printDiv.appendChild(titleElement);

    const fragenElement = document.createElement('div');
    fragenElement.innerHTML = `<strong>Antworten zu:</strong> ${content.fragen}`;
    printDiv.appendChild(fragenElement);

    printDiv.appendChild(document.createElement('hr'));
    document.body.appendChild(printDiv);
    document.body.classList.add('print-single');

    function handleAfterPrint() {
        document.body.classList.remove('print-single');
        const printDivAfter = document.getElementById('printSingleContent');
        if (printDivAfter) {
            document.body.removeChild(printDivAfter);
        }
        window.removeEventListener('afterprint', handleAfterPrint);
    }
    window.addEventListener('afterprint', handleAfterPrint);
    window.print();
}

// Funktion zur Anzeige des gespeicherten Textes (nur Fragen)
function displaySavedAnswer(content) {
    if (!savedFragenTitle || !savedFragenAnswer || !savedAnswerContainer) return;
    const titleText = parentTitle
        ? `${parentTitle}\nAntworten zu: ${assignmentSuffix}`
        : `Antworten zu: ${assignmentSuffix}`;
    savedFragenTitle.textContent = `Antworten zu: ${assignmentSuffix}`;
    savedFragenAnswer.innerHTML = content.fragen;
    savedAnswerContainer.style.display = 'block';
}

// Debounce-Funktion
function debounce(func, wait) {
    let timeout;
    return function(...args) {
        clearTimeout(timeout);
        timeout = setTimeout(() => func.apply(this, args), wait);
    };
}
const debouncedSave = debounce(saveToLocal, 2000);

// Event Listener für Textänderungen im Fragen-Editor
if (quillFragen) {
    quillFragen.on('text-change', function(delta, oldDelta, source) {
        if (source === 'user') {
            debouncedSave();
        }
    });
}

// Lade gespeicherten Inhalt und setze ihn im Quill-Editor
if (quillFragen) {
    const savedText = localStorage.getItem(STORAGE_PREFIX + assignmentId);
    if (savedText) {
        const parsedSavedText = JSON.parse(savedText);
        if (parsedSavedText.fragen) {
            quillFragen.root.innerHTML = parsedSavedText.fragen;
            console.log(`Gespeicherte Fragen für ${STORAGE_PREFIX + assignmentId} geladen`);
        }
        displaySavedAnswer(parsedSavedText);
    } else {
        console.log(`Kein gespeicherter Text für ${STORAGE_PREFIX + assignmentId} gefunden`);
    }
}

// Event Listener für den Button "Antworten als PDF drucken"
if (document.getElementById("downloadAllBtn")) {
    document.getElementById("downloadAllBtn").addEventListener('click', function() {
        const currentStorageKey = STORAGE_PREFIX + assignmentId;
        const savedText = localStorage.getItem(currentStorageKey);

        if (!savedText) {
            alert("Keine gespeicherte Antwort zum Drucken oder Speichern als PDF vorhanden.");
            console.log("Versuch, die aktuelle Antwort zu drucken, aber keine ist gespeichert");
            return;
        }
        console.log("Drucken der aktuellen Antwort wird initiiert");

        const titleText = parentTitle
            ? `${parentTitle} - Antworten zu: ${assignmentSuffix}`
            : `Antworten zu: ${assignmentSuffix}`;

        const parsedContent = JSON.parse(savedText);
        printSingleAnswer(titleText, parsedContent);
    });
}

// Event Listener für die "Alle auswählen" Checkbox (falls vorhanden)
document.getElementById("selectAll")?.addEventListener('change', function() {
    const checkboxes = document.querySelectorAll(".select-answer");
    checkboxes.forEach(cb => cb.checked = this.checked);
    toggleBulkDeleteButton();
});

document.getElementById("bulkDeleteBtn")?.addEventListener('click', bulkDeleteAnswers);

function toggleBulkDeleteButton() {
    const selected = document.querySelectorAll(".select-answer:checked").length;
    const bulkDeleteBtn = document.getElementById("bulkDeleteBtn");
    bulkDeleteBtn.disabled = selected === 0;
}

function printAllAnswers(allContent) {
    const printDiv = document.createElement('div');
    printDiv.id = 'printAllContent';
    printDiv.innerHTML = allContent;
    document.body.appendChild(printDiv);
    document.body.classList.add('print-all');

    function handleAfterPrint() {
        document.body.classList.remove('print-all');
        const printDivAfter = document.getElementById('printAllContent');
        if (printDivAfter) {
            document.body.removeChild(printDivAfter);
        }
        window.removeEventListener('afterprint', handleAfterPrint);
    }
    window.addEventListener('afterprint', handleAfterPrint);
    window.print();
}

console.log("Initialer Zustand von localStorage:");
for (let i = 0; i < localStorage.length; i++) {
    const key = localStorage.key(i);
    console.log(`${key}: ${localStorage.getItem(key)}`);
}
