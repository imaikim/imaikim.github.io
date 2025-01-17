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
            <button type="button" id="startBtn" onclick="init()">시작</button>
            <button type="button" id="stopBtn" onclick="stop()">중지</button>
        </div>
        <div id="webcam-container"></div>
        <div id="label-container"></div>
    </div>

    <!-- TensorFlow.js 및 Teachable Machine 라이브러리 로드 -->
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.3.1/dist/tf.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@0.8/dist/teachablemachine-image.min.js"></script>

    <script type="text/javascript">
        // 모델 경로 설정 (GitHub Pages 경로 확인)
        const URL = "https://imaikim.github.io/my_model/"; // 변경 필요

        let model, webcam, labelContainer, maxPredictions;
        let isRunning = false; // 루프 상태 확인용 플래그

        // 모델 로드 및 웹캠 설정
        async function init() {
            if (isRunning) return;

            try {
                // 모델 경로 설정
                const modelURL = `${URL}model.json`;
                const metadataURL = `${URL}metadata.json`;

                // 모델 및 메타데이터 로드
                model = await tmImage.load(modelURL, metadataURL);
                maxPredictions = model.getTotalClasses();

                // 웹캠 설정
                webcam = new tmImage.Webcam(350, 350, true); // (width, height, flip)
                await webcam.setup(); // 웹캠 사용 권한 요청
                await webcam.play();
                isRunning = true;

                // 웹캠 캔버스 추가
                document.getElementById("webcam-container").appendChild(webcam.canvas);

                // 레이블 컨테이너 초기화
                labelContainer = document.getElementById("label-container");
                labelContainer.innerHTML = ""; // 기존 내용 초기화
                for (let i = 0; i < maxPredictions; i++) {
                    const labelDiv = document.createElement("div");
                    labelContainer.appendChild(labelDiv);
                }

                // 버튼 상태 업데이트
                document.getElementById("startBtn").style.display = "none";
                document.getElementById("stopBtn").style.display = "inline-block";

                // 예측 루프 시작
                window.requestAnimationFrame(loop);
            } catch (error) {
                console.error("Error initializing the application:", error);
            }
        }

        // 웹캠 루프
        async function loop() {
            if (!isRunning) return;
            webcam.update(); // 웹캠 프레임 업데이트
            await predict(); // 예측 실행
            window.requestAnimationFrame(loop); // 다음 루프 실행
        }

        // 예측 함수
        async function predict() {
            const predictions = await model.predict(webcam.canvas);
            let topPrediction = { className: "", probability: 0 };

            for (let i = 0; i < maxPredictions; i++) {
                const prob = predictions[i].probability.toFixed(2) * 100;
                const className = predictions[i].className;

                // 레이블 업데이트
                const labelDiv = labelContainer.childNodes[i];
                if (labelDiv) {
                    labelDiv.innerHTML = `${className}: ${prob.toFixed(2)}%`;
                    labelDiv.style.color = "black";
                }

                // 최고 확률 예측 저장
                if (prob > topPrediction.probability) {
                    topPrediction = { className, probability: prob };
                }
            }

            // 최고 확률 레이블 강조
            labelContainer.childNodes.forEach((label) => (label.style.color = "black"));
            const topIndex = predictions.findIndex(p => p.className === topPrediction.className);
            if (labelContainer.childNodes[topIndex]) {
                labelContainer.childNodes[topIndex].style.color = "red";
            }
        }

        // 웹캠 중지 및 상태 초기화
        async function stop() {
            if (!isRunning) return;

            isRunning = false;
            await webcam.stop();

            // UI 초기화
            const webcamContainer = document.getElementById("webcam-container");
            webcamContainer.innerHTML = "";
            document.getElementById("label-container").innerHTML = "";

            // 버튼 상태 업데이트
            document.getElementById("startBtn").style.display = "inline-block";
            document.getElementById("stopBtn").style.display = "none";
        }

        // 초기 상태 설정
        window.onload = function () {
            document.getElementById("stopBtn").style.display = "none";
        };
    </script>
</body>
</html>

