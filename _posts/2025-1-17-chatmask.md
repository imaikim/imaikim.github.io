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
    <title>Mask Detection</title>
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
            margin: 5px;
        }
    </style>
</head>
<body>
<div id="container">
    <h1>마스크 착용 확인</h1>
    <div id="button-container">
        <button type="button" id="startBtn" onclick="init()">시작</button>
        <button type="button" id="stopBtn" onclick="stop()" style="display: none;">중지</button>
    </div>
    <div id="webcam-container"></div>
    <div id="label-container"></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image"></script>
<script>
    const URL = "./my_model/"; // 모델 폴더 경로

    let model, webcam, labelContainer, maxPredictions;
    let isRunning = false;

    // 웹캠과 모델 초기화
    async function init() {
        try {
            if (isRunning) {
                console.log("이미 실행 중입니다.");
                return;
            }

            isRunning = true;
            document.getElementById('startBtn').style.display = 'none';
            document.getElementById('stopBtn').style.display = 'inline';

            console.log("모델 및 메타데이터 로드 중...");
            const modelURL = `${URL}model.json`;
            const metadataURL = `${URL}metadata.json`;
            model = await tmImage.load(modelURL, metadataURL);
            maxPredictions = model.getTotalClasses();
            console.log("모델 로드 완료!");

            console.log("웹캠 설정 중...");
            webcam = new tmImage.Webcam(350, 350, true); // width, height, flip
            await webcam.setup(); // 웹캠 권한 요청
            await webcam.play();
            console.log("웹캠 설정 완료!");

            document.getElementById("webcam-container").appendChild(webcam.canvas);

            labelContainer = document.getElementById("label-container");
            labelContainer.innerHTML = '';
            for (let i = 0; i < maxPredictions; i++) {
                const labelElement = document.createElement("div");
                labelContainer.appendChild(labelElement);
            }

            console.log("애플리케이션 실행 중...");
            window.requestAnimationFrame(loop);
        } catch (error) {
            console.error("애플리케이션 초기화 중 오류 발생:", error);
            alert("초기화에 실패했습니다. 브라우저 콘솔을 확인하세요.");
            stop();
        }
    }

    async function loop() {
        if (!isRunning) return;
        webcam.update();
        await predict();
        window.requestAnimationFrame(loop);
    }

    async function predict() {
        const predictions = await model.predict(webcam.canvas);
        for (let i = 0; i < maxPredictions; i++) {
            const label = predictions[i].className;
            const probability = predictions[i].probability.toFixed(2) * 100;
            labelContainer.childNodes[i].innerHTML = `${label}: ${probability}%`;
        }
    }

    function stop() {
        isRunning = false;
        if (webcam && webcam.canvas) {
            webcam.stop();
            document.getElementById("webcam-container").innerHTML = '';
        }
        document.getElementById('startBtn').style.display = 'inline';
        document.getElementById('stopBtn').style.display = 'none';
        console.log("애플리케이션 중지");
    }
</script>
</body>
</html>


