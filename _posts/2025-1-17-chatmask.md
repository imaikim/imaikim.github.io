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
    <title>Teachable Machine Image</title>
    <style>
        #container {
            display: flex;
            flex-direction: column;
            align-items: center;
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
        <button type="button" id='startBtn' onclick="init()">시작</button>
        <button type="button" id='stopBtn' onclick="stop()">중지</button>
    </div>
    <div id="webcam-container"></div>
    <div id="label-container"></div>
</div>

<!-- TensorFlow.js and Teachable Machine -->
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.3.1/dist/tf.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@0.8/dist/teachablemachine-image.min.js"></script>

<script type="text/javascript">
    const URL = "https://imaikim.github.io/my_model/"; // GitHub 또는 다른 경로에서 모델 URL 설정

    let model, webcam, labelContainer, maxPredictions;

    var flag = false;

    // 웹캠 및 모델 초기화
    async function init() {
        var element = document.getElementById('webcam-container');
        if (element.hasChildNodes()) {
            return;
        }

        flag = true;
        const modelURL = URL + "model.json";
        const metadataURL = URL + "metadata.json";

        // 모델 및 메타데이터 로드
        try {
            model = await tmImage.load(modelURL, metadataURL);
            maxPredictions = model.getTotalClasses();
        } catch (error) {
            console.error("모델 로드 실패:", error);
            return;
        }

        const flip = true; 
        webcam = new tmImage.Webcam(350, 350, flip); 
        await webcam.setup(); 
        await webcam.play();
        window.requestAnimationFrame(loop);

        document.getElementById("webcam-container").appendChild(webcam.canvas);

        labelContainer = document.getElementById("label-container");
        for (let i = 0; i < maxPredictions; i++) { 
            labelContainer.appendChild(document.createElement("div"));
        }

        document.getElementById("startBtn").style.visibility = "hidden";
        document.getElementById("stopBtn").style.visibility = "visible";
    }

    async function loop() {
        webcam.update(); 
        await predict();
        if (flag) {
            window.requestAnimationFrame(loop);
        }
    }

    async function predict() {
        const prediction = await model.predict(webcam.canvas);
        
        if (labelContainer) {
            var topChild;
            var topProb = 0;
            var topClassName = "";

            for (let i = 0; i < maxPredictions; i++) {
                prob = prediction[i].probability * 100;
                if (prob > topProb) {
                    topChild = labelContainer.childNodes[i];
                    topProb = prob;
                    topClassName = prediction[i].className + ": " + prob.toFixed(2) + "%";
                }
                // 자식 노드가 정의되지 않은 경우를 방지
                if (labelContainer.childNodes[i]) {
                    labelContainer.childNodes[i].innerHTML = "";
                }
            }
            
            if (topChild) {
                topChild.innerHTML = topClassName;
                topChild.style.color = "white";
            }
        }
    }

    async function stop() {
        flag = false;
        webcam.stop();
        document.getElementById("webcam-container").removeChild(webcam.canvas);
        const labels = document.getElementById("label-container");
        while (labels.firstChild) {
            labels.removeChild(labels.lastChild);
        }
        document.getElementById("startBtn").style.visibility = "visible";
        document.getElementById("stopBtn").style.visibility = "hidden";
    }

    window.onload = function () {
        document.getElementById("stopBtn").style.visibility = "hidden";
    }
</script>
</body>
</html>




