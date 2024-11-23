<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>좋은 자세 판별기</title>
    <style>
        /* 전반적인 페이지 스타일 */
        body {
            font-family: 'Arial', sans-serif;
            background: linear-gradient(135deg, #f9fafb, #e5e7eb);
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            flex-direction: column;
            color: #333;
        }

        /* 제목 스타일 */
        h1 {
            font-size: 3rem;
            color: #4CAF50;
            text-align: center;
            margin-bottom: 30px;
            text-transform: uppercase;
            font-weight: bold;
        }

        /* 버튼 스타일 */
        button {
            background-color: #4CAF50;
            color: white;
            font-size: 1.4rem;
            padding: 15px 30px;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 10px 15px rgba(0, 0, 0, 0.1);
            margin-bottom: 30px;
        }

        button:hover {
            background-color: #45a049;
            transform: scale(1.05);
            box-shadow: 0 15px 30px rgba(0, 0, 0, 0.2);
        }

        button:active {
            background-color: #388e3c;
            transform: scale(1);
        }

        /* 웹캠 비디오 컨테이너 */
        #webcam-container {
            display: flex;
            justify-content: center;
            align-items: center;
            width: 250px;
            height: 250px;
            border: 5px solid #4CAF50;
            border-radius: 12px;
            background-color: #fff;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
            margin-bottom: 20px;
        }

        /* 라벨 컨테이너 스타일 */
        #label-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 15px;
        }

        /* 각 라벨의 스타일 */
        #label-container div {
            background-color: #ffffff;
            padding: 15px;
            margin: 5px;
            width: 220px;
            text-align: center;
            font-size: 1.2rem;
            color: #333;
            border-radius: 10px;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.15);
            transition: all 0.3s ease;
        }

        #label-container div:hover {
            background-color: #f1f1f1;
            transform: scale(1.05);
        }

        /* 반응형 스타일: 화면 크기에 따라 크기 조절 */
        @media (max-width: 768px) {
            h1 {
                font-size: 2.5rem;
            }

            button {
                font-size: 1.2rem;
                padding: 12px 24px;
            }

            #webcam-container {
                width: 200px;
                height: 200px;
            }

            #label-container div {
                width: 200px;
            }
        }
    </style>
</head>
<body>
    <h1>좋은 자세 판별기</h1>
    <button type="button" onclick="init()">시작</button>
    <div id="webcam-container"></div>
    <div id="label-container"></div>

    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@latest/dist/teachablemachine-image.min.js"></script>
    <script type="text/javascript">
        const URL = "https://teachablemachine.withgoogle.com/models/-3UmvuLzK/";

        let model, webcam, labelContainer, maxPredictions;
        let soundEffect = new Audio('https://www.soundjay.com/button/beep-07.wav');  // 따봉 소리

        async function init() {
            const modelURL = URL + "model.json";
            const metadataURL = URL + "metadata.json";

            model = await tmImage.load(modelURL, metadataURL);
            maxPredictions = model.getTotalClasses();

            const flip = true;
            webcam = new tmImage.Webcam(200, 200, flip);
            await webcam.setup();
            await webcam.play();
            window.requestAnimationFrame(loop);

            document.getElementById("webcam-container").appendChild(webcam.canvas);
            labelContainer = document.getElementById("label-container");
            for (let i = 0; i < maxPredictions; i++) {
                labelContainer.appendChild(document.createElement("div"));
            }
        }

        async function loop() {
            webcam.update();
            await predict();
            window.requestAnimationFrame(loop);
        }

        async function predict() {
            const prediction = await model.predict(webcam.canvas);
            for (let i = 0; i < maxPredictions; i++) {
                const classPrediction = prediction[i].className + ": " + prediction[i].probability.toFixed(2);
                labelContainer.childNodes[i].innerHTML = classPrediction;
                
                // 예측된 클래스가 1일 때 따봉 소리 출력
                if (prediction[i].className === "1" && prediction[i].probability > 0.9) { 
                    soundEffect.play(); // 소리 출력
                }
            }
        }
    </script>
</body>
</html>