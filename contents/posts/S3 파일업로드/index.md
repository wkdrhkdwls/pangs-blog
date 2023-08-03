---
title: "Next에서 WebCam으로 S3로 이미지 업로드하기"
description: "S3로 이미지 업로드하기"
date: 2023-08-03
update: 2023-08-03
tags:
  - ReactJS
  - NextJS
  - AWS
  - Blog
  - S3
  - WebCam
series: "Project"
---

이전 게시물에 이어 S3 생성 이후 이미지를 업로드 하는 방법을 작성하도록 하겠다. 카메라 기능을 이용하여 캡쳐 시 S3서버로 이미지를 업로드 시켜야 했기에, 카메라 기능을 하는 모듈이 필요했다.<br/>
때문에 ReactWebcam을 사용해서 카메라 기능을 구현하기로 하였다.

## 1. Webcam 설치

> npm install react-webcam
> ReactWebcam을 먼저 설치한다.

## 2. Webcam.js 세팅

```JavaScript
  const webcamRef = useRef(null);
  const [isCameraOpen, setIsCameraOpen] = useState(false);
  const router = useRouter();

  const captureAndUpload = () => {
    const imageSrc = webcamRef.current.getScreenshot();
    router.push(`/camera`);
  };

  const startCapture = () => {
    setIsCameraOpen(true);
    webcamRef.current?.startCapture();
  };

  const stopCapture = () => {
    setIsCameraOpen(false);
    const videoElement = webcamRef.current.video;
    videoElement.pause();
  };

  const videoConstraints = {
    width: 640,
    height: 480,
    facingMode: "environment",
  };
  return (
    <div className="flex flex-col items-center justify-center">
      {!isCameraOpen && (
        <button
          onClick={startCapture}
          className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
        >
          <FaCamera size={32} color="white" />
        </button>
      )}
      {isCameraOpen && (
        <>
          <div className="relative">
            <Webcam
              audio={false}
              ref={webcamRef}
              videoConstraints={videoConstraints}
              className="rounded-md"
            />
            <div className="absolute top-0 left-0 right-0 bottom-0 grid grid-cols-3 grid-rows-3 gap-0">
              {Array.from(Array(9).keys()).map((index) => (
                <div
                  key={index}
                  className="border border-gray-500"
                  style={{
                    gridColumn: `span 1`,
                    gridRow: `span 1`,
                  }}
                />
              ))}
            </div>
          </div>
          <div className="mt-4">
            <button
              onClick={captureAndUpload}
              className="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded mr-2"
            >
              캡쳐하기
            </button>
            <button
              onClick={stopCapture}
              className="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded"
            >
              <FaTimes size={20} />
            </button>
          </div>
        </>
      )}
    </div>
  );
```

이미지 캡처 후 페이지를 이동하여 해당 약품의 정보를 보여줄 것이기 떄문에 `next/router`를 사용하고, 모바일 환경시 카메라 후면을 활용할 것이기에 `facingMode: "environment"`와 같이 설정한다.<br/>

## 3. S3로 파일 업로드하기

Webcam의 설정이 끝났으면 본격적으로 S3로 파일을 업로드하도록 코드를 설정한다.

```JavaScript
const uploadToS3 = async (photo) => {
    const s3 = new AWS.S3({
      accessKeyId: process.env.AWS_ACCESS_KEY_ID,
      secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
    });
    const params = {
      Bucket: process.env.AWS_BUCKET_NAME,
      Key: `photo-${Date.now()}.jpg`,
      Body: photo,
      ContentType: "image/jpg",
    };
    try {
      await s3.upload(params).promise();
      console.log("Successfully uploaded photo to S3.");
    } catch (error) {
      console.error("Error uploading photo to S3:", error);
    }
  };
```

S3 서버에 올릴 `uploadToS3`를 만든다. 이떄 accessKeyId, secretAccessKey, BUCKET_NAME을 적어주어야한다. 그러나 이를 그대로 작성하여 깃허브에 올린다면 보안상 문제가 있기 떄문에 이들을 env환경으로 빼주고 gitignore에도 추가하여 준다.

```JSON
AWS_ACCESS_KEY_ID=xxxxxxxxxx
AWS_SECRET_ACCESS_KEY=xxxxxxxxxxx
AWS_BUCKET_NAME=xxxxxxxx
```

와 같이 설정한 후, 이를 `process.env.변수명`으로 불러와 사용하도록한다. 그러나 이때 문제가 하나 발생했는데, 분명 제대로 입력했음에도 불구하고 동작을 하지 않았다. 구글링 결과 Next에서는 env변수명 앞에 NEXT\_를 붙여줘야한다고 한다.

```JavaScript
const uploadToS3 = async (photo) => {
    const s3 = new AWS.S3({
      accessKeyId: process.env.NEXT_PUBLIC_AWS_ACCESS_KEY_ID,
      secretAccessKey: process.env.NEXT_PUBLIC_AWS_SECRET_ACCESS_KEY,
    });
    const params = {
      Bucket: process.env.NEXT_PUBLIC_AWS_BUCKET_NAME,
      Key: `photo-${Date.now()}.jpg`,
      Body: photo,
      ContentType: "image/jpg",
    };
    try {
      await s3.upload(params).promise();
      console.log("Successfully uploaded photo to S3.");
    } catch (error) {
      console.error("Error uploading photo to S3:", error);
    }
  };
```

때문에 다음과 같이 바꿔주면 제대로 작동하는 것을 확인할 수 있다.

### 3-1. 이미지 파일데이터 압축하기

위와 같은 형식으로 보내면 url의 길이가 매우 길어지는 것을 확인할 수 있는데 이를

```JavaScript
const dataURLtoBlob = (dataURL) => {
    const arr = dataURL.split(",");
    const mime = arr[0].match(/:(.*?);/)[1];
    const bstr = atob(arr[1]);
    let n = bstr.length;
    const u8arr = new Uint8Array(n);
    while (n--) {
      u8arr[n] = bstr.charCodeAt(n);
    }
    return new Blob([u8arr], { type: mime });
  };

  const captureAndUpload = () => {
    const imageSrc = webcamRef.current.getScreenshot();
    const blob = dataURLtoBlob(imageSrc);
    uploadToS3(blob);
    router.push(`/camera`);
  };
```

와 같이 수정해준다.

## 4. 최종 코드

```JavaScript
import Webcam from "react-webcam";
// import AWS from "aws-sdk";
import { useRef, useState } from "react";
import { FaCamera, FaTimes } from "react-icons/fa";
import { useRouter } from "next/router";

const CaptureImage = () => {
  const webcamRef = useRef(null);
  const [isCameraOpen, setIsCameraOpen] = useState(false);
  const router = useRouter();

  const dataURLtoBlob = (dataURL) => {
    const arr = dataURL.split(",");
    const mime = arr[0].match(/:(.*?);/)[1];
    const bstr = atob(arr[1]);
    let n = bstr.length;
    const u8arr = new Uint8Array(n);
    while (n--) {
      u8arr[n] = bstr.charCodeAt(n);
    }
    return new Blob([u8arr], { type: mime });
  };

  const captureAndUpload = () => {
    const imageSrc = webcamRef.current.getScreenshot();
    const blob = dataURLtoBlob(imageSrc);
    uploadToS3(blob);
    router.push(`/camera`);
  };

  const uploadToS3 = async (photo) => {
    const s3 = new AWS.S3({
      accessKeyId: process.env.NEXT_PUBLIC_AWS_ACCESS_KEY_ID,
      secretAccessKey: process.env.NEXT_PUBLIC_AWS_SECRET_ACCESS_KEY,
    });
    const params = {
      Bucket: process.env.NEXT_PUBLIC_AWS_BUCKET_NAME,
      Key: `photo-${Date.now()}.jpg`,
      Body: photo,
      ContentType: "image/jpg",
    };
    try {
      await s3.upload(params).promise();
      console.log("Successfully uploaded photo to S3.");
    } catch (error) {
      console.error("Error uploading photo to S3:", error);
    }
  };

  const startCapture = () => {
    setIsCameraOpen(true);
    webcamRef.current?.startCapture();
  };

  const stopCapture = () => {
    setIsCameraOpen(false);
    const videoElement = webcamRef.current.video;
    videoElement.pause();
  };

  const videoConstraints = {
    width: 640,
    height: 480,
    facingMode: "environment",
  };

  return (
    <div className="flex flex-col items-center justify-center">
      {!isCameraOpen && (
        <button
          onClick={startCapture}
          className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
        >
          <FaCamera size={32} color="white" />
        </button>
      )}
      {isCameraOpen && (
        <>
          <div className="relative">
            <Webcam
              audio={false}
              ref={webcamRef}
              videoConstraints={videoConstraints}
              className="rounded-md"
            />
            <div className="absolute top-0 left-0 right-0 bottom-0 grid grid-cols-3 grid-rows-3 gap-0">
              {Array.from(Array(9).keys()).map((index) => (
                <div
                  key={index}
                  className="border border-gray-500"
                  style={{
                    gridColumn: `span 1`,
                    gridRow: `span 1`,
                  }}
                />
              ))}
            </div>
          </div>
          <div className="mt-4">
            <button
              onClick={captureAndUpload}
              className="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded mr-2"
            >
              캡쳐하기
            </button>
            <button
              onClick={stopCapture}
              className="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded"
            >
              <FaTimes size={20} />
            </button>
          </div>
        </>
      )}
    </div>
  );
};

export default CaptureImage;
```
