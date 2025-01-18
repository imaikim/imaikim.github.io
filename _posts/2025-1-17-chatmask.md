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
    <!--문서의 문자 인코딩을 UTF-8로 설정.
    화면 뷰포트를 모바일 기기와 데스크톱에서 적절히 표시되도록 설정.
    문서의 제목을 "Mask Detection"으로 지정.-->

    <style>
        #container {
        display: flex;
        flex-direction: column;
        align-items: center;
        }
        /* #container는 플렉스박스를 사용하여 자식 요소들을 수직 방향으로 정렬하고, 가운데 정렬 */

        #label-container {
            margin-top: 20px;
            font-size: 24px;
            font-weight: bold;
        }

        #video-container {
            /*margin-top: 100px; 조절가능*/
            width: 400px;
            height: auto;
            background-color: black;
            display: none; /* 초기 상태에서 숨김 */
        }

        video {
            width: 100%;
            height: 100%;
        }
        /* <video> 태그는 #video-container 크기에 맞춰 전체 크기로 확장. */

        #button-container {
            margin-top: 20px;
            display: flex;
            gap: 20px;
        }
        /* 버튼 영역을 플렉스박스로 배치하며, 버튼 간의 간격을 20px로 설정 */

        button {
            padding: 10px 20px;
            font-size: 16px;
            color: white;
            background-color: #007bff;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }

        button:disabled {
            background-color: #555;
        }
    </style>
</head>
<body>
<div id="container">
    <div id="label-container">마스크 착용 여부</div>
    <div id="video-container">
        <video id="webcam" autoplay></video>
    </div>
    <div id="button-container">
        <button id="startBtn">시작</button>
        <button id="stopBtn" disabled>정지</button>
    </div>
</div>

    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.3.1/dist/tf.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@0.8/dist/teachablemachine-image.min.js"></script>

    <script>
        const URL = "https://imaikim.github.io/my_model/";  // 모델 URL 정의
        let model, maxPredictions; // 변수 초기화

        // 모델 로드 함수
        async function loadModel() {
            // 비동기 함수로 정의. 이 함수는 내부의 비동기 작업(await)을 처리하며 호출 시 Promise를 반환
            const modelURL = URL + "model.json";
            // modelURL 변수에 모델 파일(model.json)의 경로를 저장. URL은 외부에서 정의된 기본 경로
            const metadataURL = URL + "metadata.json"; // metadataURL 변수에 모델 메타데이터 파일(metadata.json)의 경로를 저장
            model = await tmImage.load(modelURL, metadataURL); // Teachable Machine의 tmImage.load 메서드를 호출하여 모델과 메타데이터를 로드
            // await를 사용하여 작업 완료 후 결과(로드된 모델 객체)를 model 변수에 저장
            maxPredictions = model.getTotalClasses(); // 모델의 클래스(예측 가능한 분류) 수를 가져와 maxPredictions 변수에 저장
        }

        // 웹캠 시작 함수
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

        // 실시간 예측 루프
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

                // 결과를 화면 상단에 표시
                const label = document.getElementById('label-container');
                label.innerHTML = resultText;

                await new Promise(resolve => setTimeout(resolve, 100)); // 예측 주기 조정
            }
        }

        // 웹캠 정지 함수
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
            document.getElementById('label-container').innerHTML = "마스크 착용 여부";
            document.getElementById('startBtn').disabled = false;
            document.getElementById('stopBtn').disabled = true;
        }

        // 버튼 이벤트 핸들러
        document.getElementById('startBtn').addEventListener('click', async () => {
            if (!model) await loadModel();
            await startWebcam();
        });

        document.getElementById('stopBtn').addEventListener('click', stopWebcam);
    </script>
</body>
</html>





































