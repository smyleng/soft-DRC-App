const scanBtn = document.getElementById('scanBtn');
const cameraBtn = document.getElementById('cameraBtn');
const imageInput = document.getElementById('imageInput');
const output = document.getElementById('output');

function recognizeTextFromImage(imageSrc) {
    output.textContent = "Traitement en cours...";
    Tesseract.recognize(
        imageSrc,
        'fra',
        { logger: m => console.log(m) }
    ).then(({ data: { text } }) => {
        output.textContent = text || "Aucun texte détecté.";
    }).catch(() => {
        output.textContent = "Erreur lors de la reconnaissance.";
    });
}

// Scanner une image uploadée
scanBtn.addEventListener('click', () => {
    const file = imageInput.files[0];
    if (!file) {
        alert("Choisis une image d'abord !");
        return;
    }
    const reader = new FileReader();
    reader.onload = () => {
        recognizeTextFromImage(reader.result);
    };
    reader.readAsDataURL(file);
});

// Prendre une photo avec la caméra
cameraBtn.addEventListener('click', () => {
    const video = document.createElement('video')const canvas = document.createElement('canvas');
    const context = canvas.getContext('2d');

    navigator.mediaDevices.getUserMedia({ video: true }).then(stream => {
        video.srcObject = stream;
        video.play();

        alert("Clique sur OK, puis prends la photo avec la caméra (la vidéo va s'ouvrir dans une nouvelle fenêtre).");

        // Ouvrir la vidéo dans une nouvelle fenêtre pour que l'utilisateur puisse voir la caméra
        const videoWindow = window.open('', 'Camera', 'width=640,height=480');
        videoWindow.document.body.appendChild(video);

        // Ajouter un bouton pour capturer la photo dans cette nouvelle fenêtre
        const snapBtn = videoWindow.document.createElement('button');
        snapBtn.textContent = "Prendre la photo";
        videoWindow.document.body.appendChild(snapBtn);

        snapBtn.onclick = () => {
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            stream.getTracks().forEach(track => track.stop()); // Arrêter la caméra
            videoWindow.close();

            const imageData = canvas.toDataURL('image/png');
            recognizeTextFromImage(imageData);
        };
    }).catch(() => {
