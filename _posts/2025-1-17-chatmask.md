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
        const URL = "https://imaikim.github.io/my_model/";  // 모델 URL 정의 본인 github레포지토리 주소
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
            // 비동기 함수로 정의. 웹캠 스트림을 비동기로 처리.
            const videoElement = document.getElementById('webcam');
            // videoElement 변수에 <video> 요소를 참조하여 저장.
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: true });
                // getUserMedia 메서드를 호출하여 사용자 장치의 웹캠 스트림을 요청.
                // { video: true }는 비디오만 활성화하겠다는 설정.
                // await를 사용하여 스트림 반환 후 결과를 stream 변수에 저장.
                videoElement.srcObject = stream; // <video> 요소의 srcObject 속성에 스트림을 연결해 웹캠의 실시간 화면을 표시.
                videoElement.style.display = 'block'; // <video> 요소를 화면에 표시.
                document.getElementById('video-container').style.display = 'block';
                document.getElementById('label-container').style.display = 'block';
                // #video-container와 #label-container를 화면에 표시.
                document.getElementById('startBtn').disabled = true;
                document.getElementById('stopBtn').disabled = false;
                //"시작" 버튼을 비활성화하고, "정지" 버튼을 활성화
                predictLoop(videoElement);
                // predictLoop 함수 호출, 비디오를 기반으로 실시간 예측 작업 시작.
            } catch (err) {
                alert('웹캠 접근이 거부되었습니다. 권한을 확인해주세요.');
            }
            // try 블록에서 예외가 발생하면 catch 블록으로 이동.
            // 웹캠 접근이 실패한 경우 사용자에게 경고 메시지를 표시.
        }

        // 실시간 예측 루프
        async function predictLoop(videoElement) {
            // 비동기 함수 정의. 실시간 예측 루프를 구현.
            while (!document.getElementById('stopBtn').disabled) {
                // "정지" 버튼이 비활성화되지 않은 동안 반복. 즉, 사용자가 "정지" 버튼을 누르지 않는 한 계속 실행.
                const prediction = await model.predict(videoElement);
                // Teachable Machine 모델의 predict 메서드를 호출하여, 비디오 요소에서 실시간 프레임을 받아 예측 수행. 반환값은 예측 결과 배열.
                let highestProb = 0;
                let resultText = "";
                // 가장 높은 확률(highestProb)과 결과 텍스트(resultText)를 초기화.
                prediction.forEach(pred => {
                // 각 예측 결과(pred)에 대해 반복.
                    if (pred.probability > highestProb) {
                        highestProb = pred.probability;
                        resultText = `${pred.className}: ${(highestProb * 100).toFixed(2)}%`;
                    }
                    // 현재 예측(pred)의 확률이 가장 높은 확률(highestProb)보다 크면 업데이트.
                    // 클래스 이름과 확률을 포함한 결과 텍스트(resultText)를 업데이트.
                });
                // 반복 종료 후 highestProb와 resultText는 가장 높은 확률을 가진 예측을 나타냄.

                // 결과를 화면 상단에 표시
                const label = document.getElementById('label-container');
                label.innerHTML = resultText;
                // 예측 결과 텍스트(resultText)를 화면 상단의 #label-container에 표시.

                await new Promise(resolve => setTimeout(resolve, 100)); // 예측 주기 조정
                // 루프 실행을 100ms 대기. 프레임 간 간격 조정.
            }
            // 사용자가 "정지" 버튼을 누를 때까지 루프 계속.
        }

        // 웹캠 정지 함수
        function stopWebcam() {
        // 웹캠 스트림 정지 및 화면 초기화를 처리하는 함수.
            const videoElement = document.getElementById('webcam');
            const stream = videoElement.srcObject;
            // <video> 요소와 연결된 스트림(srcObject)을 가져옴.

            if (stream) {
                const tracks = stream.getTracks();
                tracks.forEach(track => track.stop());
            }
            // 스트림이 있을 경우, 모든 트랙(오디오/비디오)을 가져와 정지(stop()) 처리.

            videoElement.srcObject = null;
            videoElement.style.display = 'none';
            // <video> 요소의 스트림 연결 해제 및 화면에서 숨김.

            document.getElementById('video-container').style.display = 'none';
            document.getElementById('label-container').innerHTML = "마스크 착용 여부";
            // #video-container와 결과 텍스트(label-container)를 초기화하여 화면에서 숨김 또는 기본 상태로 복구.
            document.getElementById('startBtn').disabled = false;
            document.getElementById('stopBtn').disabled = true;
            // "시작" 버튼 활성화, "정지" 버튼 비활성화.
        }

        // 버튼 이벤트 핸들러
        document.getElementById('startBtn').addEventListener('click', async () => {
        // "시작" 버튼 클릭 시 호출할 이벤트 핸들러 등록.
            if (!model) await loadModel();
            // 모델이 로드되지 않았다면 loadModel 함수 호출.
            await startWebcam();
            // 웹캠 스트림 시작.
        });

        document.getElementById('stopBtn').addEventListener('click', stopWebcam);
        // "정지" 버튼 클릭 시 stopWebcam 함수 호출.
    </script>
</body>
</html>





































