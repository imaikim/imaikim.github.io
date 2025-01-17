---
layout: single
title:  "마스크착용확인유무딥러닝chat!!"
categories: coding
tag: [python , blog]
# toc: true
author_profile: false
# sidebar:
#   nav: docs
---



# 시작버튼을 클릭하세요

<!-- <!DOCTYPE html> -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Teachable Machine Webcam</title>
    <style>
        #container {
            display: flex;
            flex-direction: column;
            align-items: center;
            margin-top: 20px;
        }

        #webcam-container, #label-container {
            margin: 10px;
        }

        #button-container {
            display: flex;
            justify-content: space-around;
            width: 100%;
        }

        #startBtn, #stopBtn {
            padding: 10px;
        }
    </style>
</head>
<body>
<div id="container">
    <div id="button-container">
        <button type="button" id="startBtn" onclick="init()">Start Webcam</button>
        <button type="button" id="stopBtn" onclick="stop()" style="visibility: hidden;">Stop Webcam</button>
    </div>
    <div id="webcam-container"></div>
    <div id="label-container"></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image"></script>

<script type="text/javascript">
    const URL = "YOUR_MODEL_URL/"; // Replace with the correct model path

    let model, webcam, labelContainer, maxPredictions;
    let isRunning = false;

    async function init() {
        if (isRunning) return;

        const modelURL = URL + "model.json";
        const metadataURL = URL + "metadata.json";

        // Load the model and metadata
        model = await tmImage.load(modelURL, metadataURL);
        maxPredictions = model.getTotalClasses();

        // Initialize the webcam
        const flip = true; // Flip the webcam horizontally
        webcam = new tmImage.Webcam(350, 350, flip); // Width, height, flip
        await webcam.setup(); // Request webcam access
        await webcam.play();
        window.requestAnimationFrame(loop);

        document.getElementById("webcam-container").appendChild(webcam.canvas);

        // Prepare label container
        labelContainer = document.getElementById("label-container");
        labelContainer.innerHTML = "";
        for (let i = 0; i < maxPredictions; i++) {
            labelContainer.appendChild(document.createElement("div"));
        }

        document.getElementById("startBtn").style.visibility = "hidden";
        document.getElementById("stopBtn").style.visibility = "visible";

        isRunning = true;
    }

    async function loop() {
        if (!isRunning) return;

        webcam.update(); // Update webcam frame
        await predict();
        window.requestAnimationFrame(loop);
    }

    async function predict() {
        const predictions = await model.predict(webcam.canvas);

        predictions.forEach((prediction, index) => {
            const classPrediction =
                `${prediction.className}: ${(prediction.probability * 100).toFixed(2)}%`;
            labelContainer.childNodes[index].innerText = classPrediction;
        });
    }

    async function stop() {
        isRunning = false;
        webcam.stop();
        document.getElementById("webcam-container").innerHTML = "";
        labelContainer.innerHTML = "";

        document.getElementById("startBtn").style.visibility = "visible";
        document.getElementById("stopBtn").style.visibility = "hidden";
    }
</script>
</body>
</html>
