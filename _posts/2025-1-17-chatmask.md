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
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mask Detection</title>
    <style>
        #container {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            width: 100%;
            height: 100vh;
        }

        #button-container {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-bottom: 20px;
        }

        button {
            padding: 10px 20px;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }

        button:disabled {
            background-color: #cccccc;
        }

        #video-container {
            position: relative;
            width: 350px;
            height: 350px;
            background-color: black;
            display: none; /* 초기 상태에서 숨김 */
        }

        #label-container {
            margin-top: 20px;
            font-size: 20px;
            color: white;
            text-align: center;
            display: none; /* 초기 상태에서 숨김 */
        }
    </style>
</head>
<body>
<div id="container">
    <div id="button-container">
        <button id="startBtn">시작</button>
        <button id="stopBtn" disabled>정지</button>
    </div>
    <div id="video-container">
        <video id="webcam" autoplay></video>
    </div>
    <div id="label-container"></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.3.1/dist/tf.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@0.8/dist/teachablemachine-image.min.js"></script>

<script>
    const URL = "https://imaikim.github.io/my_model/";  // 모델 URL
    let model, webcam, labelContainer, maxPredictions;

    async function loadModel() {
        const modelURL = URL + "model.json";
        const metadataURL = URL + "metadata.json";
        model = await tmImage.load(modelURL, metadataURL);
        maxPredictions = model.getTotalClasses();
    }

    async function startWebcam() {
        const videoElement = document.getElementById('webcam');
        try {
            const stream = await navigator.mediaDevices.getUserMedia({ video: true });
            videoElement.srcObject = stream;
            videoElement.style.display = 'block';

            document.getElementById('video-container').style.display = 'block';
            document.getElementById('label-container').style.display = 'block';
            document.getElementById('startBtn').disabled = true;
            document.getElementById('stopBtn').disabled = false;

            predictLoop(videoElement);
        } catch (err) {
            alert('웹캠 접근이 거부되었습니다. 권한을 확인해주세요.');
        }
    }

    async function predictLoop(videoElement) {
        while (!document.getElementById('stopBtn').disabled) {
            const prediction = await model.predict(videoElement);
            let highestProb = 0;
            let resultText = "";

            prediction.forEach(pred => {
                if (pred.probability > highestProb) {
                    highestProb = pred.probability;
                    resultText = `${pred.className}: ${(highestProb * 100).toFixed(2)}%`;
                }
            });

            const label = document.getElementById('label-container');
            label.innerHTML = resultText;

            await new Promise(resolve => setTimeout(resolve, 100)); // 예측 주기 조정
        }
    }

    function stopWebcam() {
        const videoElement = document.getElementById('webcam');
        const stream = videoElement.srcObject;

        if (stream) {
            const tracks = stream.getTracks();
            tracks.forEach(track => track.stop());
        }

        videoElement.srcObject = null;
        videoElement.style.display = 'none';

        document.getElementById('video-container').style.display = 'none';
        document.getElementById('label-container').style.display = 'none';
        document.getElementById('label-container').innerHTML = '';

        document.getElementById('startBtn').disabled = false;
        document.getElementById('stopBtn').disabled = true;
    }

    document.getElementById('startBtn').addEventListener('click', async () => {
        if (!model) await loadModel();
        await startWebcam();
    });

    document.getElementById('stopBtn').addEventListener('click', stopWebcam);
</script>
</body>
</html>


































