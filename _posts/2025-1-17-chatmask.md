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
        body {
            font-family: Arial, sans-serif;
            background-color: #282c34;
            color: white;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }

        #container {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            width: 100%;
            max-width: 500px;
            text-align: center;
        }

        #webcam-container {
            width: 100%;
            height: 350px;
            margin: 20px 0;
            position: relative;
            border: 2px solid #fff;
        }

        video {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }

        #button-container {
            display: flex;
            justify-content: space-between;
            width: 100%;
        }

        #startBtn, #stopBtn {
            padding: 10px 20px;
            font-size: 16px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin: 10px;
        }

        #stopBtn {
            background-color: #f44336;
        }

        #label-container {
            background-color: rgba(0, 0, 0, 0.6);
            padding: 10px;
            width: 100%;
            font-size: 18px;
        }

        #stopBtn {
            display: none;
        }
    </style>
</head>
<body>

<div id="container">
    <div id="button-container">
        <button type="button" id="startBtn">시작</button>
        <button type="button" id="stopBtn">중지</button>
    </div>
    <div id="webcam-container"></div>
    <div id="label-container"></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.3.1/dist/tf.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@0.8/dist/teachablemachine-image.min.js"></script>

<script>
    const URL = "https://imaikim.github.io/my_model/";  // 여기에 실제 모델 URL을 입력하세요
    let model, webcam, labelContainer, maxPredictions;

    // 웹캠을 시작하는 함수
    function startWebcam() {
        navigator.mediaDevices.getUserMedia({ video: true })
            .then(function(stream) {
                const videoElement = document.createElement('video');
                videoElement.srcObject = stream;
                videoElement.play();

                // 웹캠 영상 화면에 표시
                document.getElementById('webcam-container').appendChild(videoElement);

                // 모델 로드 및 웹캠 처리
                loadModelAndStart(videoElement);
            })
            .catch(function(error) {
                console.error('웹캠을 열 수 없습니다.', error);
                alert('웹캠에 접근할 수 없습니다. 브라우저에서 권한을 확인해주세요.');
            });
    }

    // 모델을 로드하고 예측을 시작하는 함수
    async function loadModelAndStart(videoElement) {
        // 모델 로드
        const modelURL = URL + "model.json";
        const metadataURL = URL + "metadata.json";
        model = await tmImage.load(modelURL, metadataURL);
        maxPredictions = model.getTotalClasses();

        // 웹캠 스트림 설정
        const flip = true;
        webcam = new tmImage.Webcam(350, 350, flip); // width, height, flip
        await webcam.setup();
        await webcam.play();

        // 캔버스를 웹페이지에 표시
        document.getElementById("webcam-container").appendChild(webcam.canvas);

        // 레이블을 표시할 컨테이너 설정
        labelContainer = document.getElementById("label-container");
        for (let i = 0; i < maxPredictions; i++) {
            labelContainer.appendChild(document.createElement("div"));
        }

        // 예측을 시작하는 루프
        window.requestAnimationFrame(loop);

        // 버튼 상태 변경
        document.getElementById('startBtn').style.display = "none";
        document.getElementById('stopBtn').style.display = "inline-block";
    }

    // 웹캠으로부터 이미지를 업데이트하고 예측을 진행하는 함수
    async function loop() {
        webcam.update(); // 웹캠의 최신 이미지를 가져옴
        await predict(); // 예측 진행
        window.requestAnimationFrame(loop);
    }

    // 모델을 사용하여 예측을 수행하는 함수
    async function predict() {
        const prediction = await model.predict(webcam.canvas);

        let topProb = 0;
        let topClassName = "";

        for (let i = 0; i < maxPredictions; i++) {
            const prob = prediction[i].probability * 100;

            if (prob > topProb) {
                topProb = prob;
                topClassName = prediction[i].className + ": " + prob.toFixed(2) + "%";
            }

            // 레이블 업데이트
            labelContainer.childNodes[i].innerHTML = "";
        }

        // 가장 높은 확률을 가진 클래스 표시
        const topChild = labelContainer.childNodes[0]; // 예시로 첫 번째 자식만 사용
        topChild.innerHTML = topClassName;
        topChild.style.color = "white";
    }

    // 시작 버튼 클릭 시 웹캠 시작
    document.getElementById('startBtn').addEventListener('click', startWebcam);

    // 중지 버튼 클릭 시 웹캠 중지
    document.getElementById('stopBtn').addEventListener('click', function() {
        webcam.stop();
        document.getElementById('webcam-container').innerHTML = '';
        labelContainer.innerHTML = '';
        document.getElementById('startBtn').style.display = "inline-block";
        document.getElementById('stopBtn').style.display = "none";
    });

    window.onload = function () {
        document.getElementById('stopBtn').style.display = "none";
    }
</script>

</body>
</html>


















