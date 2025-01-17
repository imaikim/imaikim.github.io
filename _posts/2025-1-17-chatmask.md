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
            justify-content: center;
        }

        #webcam-container {
            position: relative;
            margin: 10px;
            width: 350px;
            height: 350px;
            background-color: black;
        }

        #label-container {
            margin: 20px;
            color: white;
            font-size: 20px;
            text-align: center;
        }

        #button-container {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-bottom: 10px;
        }

        button {
            padding: 10px 20px;
            background-color: #ff6600;
            border: none;
            border-radius: 5px;
            color: white;
            cursor: pointer;
        }

        button:disabled {
            background-color: #ccc;
        }

        /* 오렌지색 박스 스타일 */
        #orange-box {
            background-color: #ff6600;
            width: 100%;
            height: 50px;
            display: flex;
            justify-content: center;
            align-items: center;
            color: white;
            font-size: 18px;
            visibility: hidden; /* 초기에 숨김 */
        }
    </style>
</head>
<body>
<div id="container">
    <div id="button-container">
        <button type="button" id='startBtn'>시작</button>
        <button type="button" id='stopBtn' disabled>중지</button>
    </div>
    <div id="webcam-container"></div>
    <div id="label-container">마스크 인식 결과가 여기에 표시됩니다.</div>
    <div id="orange-box">마스크 인식 중...</div> <!-- 오렌지색 박스 -->
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

                // 버튼 상태 변경
                document.getElementById('startBtn').disabled = true;
                document.getElementById('stopBtn').disabled = false;
                document.getElementById('orange-box').style.visibility = 'hidden'; // 오렌지색 박스 숨김
                document.getElementById('label-container').style.visibility = 'hidden'; // 텍스트 박스 숨김
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

        // 예측을 시작하는 루프
        window.requestAnimationFrame(loop);
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
        }

        // 가장 높은 확률을 가진 클래스를 레이블로 표시
        document.getElementById("label-container").innerHTML = topClassName;
    }

    // 시작 버튼 클릭 시 웹캠 시작
    document.getElementById('startBtn').addEventListener('click', startWebcam);

    // 중지 버튼 클릭 시 웹캠 중지
    document.getElementById('stopBtn').addEventListener('click', function() {
        webcam.stop();
        document.getElementById('webcam-container').innerHTML = '';
        document.getElementById('orange-box').style.visibility = 'visible'; // 오렌지색 박스 보이기
        document.getElementById('label-container').style.visibility = 'visible'; // 텍스트 박스 보이기
        document.getElementById('label-container').innerHTML = '마스크 인식이 중지되었습니다.';
        document.getElementById('startBtn').disabled = false;
        document.getElementById('stopBtn').disabled = true;
    });

    window.onload = function () {
        document.getElementById('stopBtn').disabled = true;
        document.getElementById('orange-box').style.visibility = 'visible'; // 초기에는 오렌지색 박스 보이기
    }
</script>
</body>
</html>























