//<화소처리>


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script>
        // 전역 변수 (중요한 변수들)
        var inCanvas, inCtx, outCanvas,outCtx;  // 입력 캔버스 관련
        var inFile, inImageArray, outImageArray;  // 입력 파일 및 배열
        var inWidth, inHeight, outWidth, outHeight;  // 입력 영상의 폭과 높이
        var inPaper, outPaper; // 캔버스에는 한점한점이 안찍힘. 대신 캔버스에 종이를 붙임.
        // 초기화 함수 (= 생성자 함수 개념)
        function init() {  
            inCanvas = document.getElementById('inCanvas');
            inCtx = inCanvas.getContext('2d');
            outCanvas = document.getElementById('outCanvas');
            outCtx = outCanvas.getContext('2d');
        
        }
        function readRawImage() {
            inFile = document.getElementById('selectFile').files[0];
            // 중요! 코드 (영상의 크기를 파악)
            inWidth = inHeight = Math.sqrt(inFile.size);
            // 입력 2차원 배열을 준비
            inImageArray = new Array(inHeight); // 256짜리 1차원 배열
            for(var i=0; i<inHeight; i++) 
                inImageArray[i] = new Array(inWidth);
            // 캔버스 크기를 결정
            inCanvas.width = inWidth;
            inCanvas.height = inHeight;
            // RAW 파일  --> 2차원 배열
            var reader = new FileReader();
            reader.readAsBinaryString(inFile);
            reader.onload = function () {
                var bin = reader.result; // 파일을 덩어리(bin)로 읽었음
                // 덩어리(bin)에서 한점한점씩 뽑아서, 배열에 넣기
                for(var i=0; i<inHeight; i++) {
                    for(var k=0; k<inWidth; k++) {
                        // 0,0  0,1  0,2 ...... 0,255
                        // 1,0  1,1, 1,2 .......1,255
                        // ....
                        // 255,0  255,1 ....... 255,255
                        var sPixel = (i * inHeight + k);
                        var ePixel = (i * inHeight + k) + 1;
                        inImageArray[i][k] = bin.slice(sPixel,ePixel); // 1개픽셀-->배열
                    }
                }
                // 화면에 출력하기 (사람용)
                inPaper = inCtx.createImageData(inHeight, inWidth); //종이 붙였음.
                for(var i=0; i<inHeight; i++) {
                    for (var k=0; k<inWidth; k++) {
                        var charValue = inImageArray[i][k].charCodeAt(0); // 깨진문자를 숫자로.
                        inPaper.data[(i*inWidth + k) * 4 + 0] = charValue; // R
                        inPaper.data[(i*inWidth + k) * 4 + 1] = charValue; // G
                        inPaper.data[(i*inWidth + k) * 4 + 2] = charValue; // B
                        inPaper.data[(i*inWidth + k) * 4 + 3] = 255; // Alpha
                    }
                }
                inCtx.putImageData(inPaper,0,0);
            }
        }        
        for(var i=0; i<outHeight; i++) {
                for (var k=0; k<outWidth; k++) {
                    var charValue = outImageArray[i][k].charCodeAt(0); // 깨진문자를 숫자로.
                    outPaper.data[(i*outWidth + k) * 4 + 0] = charValue; // R
                    outPaper.data[(i*outWidth + k) * 4 + 1] = charValue; // G
                    outPaper.data[(i*outWidth + k) * 4 + 2] = charValue; // B
                    outPaper.data[(i*outWidth + k) * 4 + 3] = 255; // Alpha
                }
        }
///////  영상 처리 함수 모음 //////////
function printOutImage() {
              // 캔버스 크기를 결정
            outCanvas.width = outWidth;
            outCanvas.height = outHeight;
            outPaper = outCtx.createImageData(outHeight, outWidth); //종이 붙였음.
            for(var i=0; i<outHeight; i++) {
                for (var k=0; k<outWidth; k++) {
                    var charValue = outImageArray[i][k].charCodeAt(0); // 깨진문자를 숫자로.
                    outPaper.data[(i*outWidth + k) * 4 + 0] = charValue; // R
                    outPaper.data[(i*outWidth + k) * 4 + 1] = charValue; // G
                    outPaper.data[(i*outWidth + k) * 4 + 2] = charValue; // B
                    outPaper.data[(i*outWidth + k) * 4 + 3] = 255; // Alpha
                }
            }
            outCtx.putImageData(outPaper,0,0);
        }

        function addImage() {
            // (중요!) 출력 영상의 크기를 결정... 알고리즘에 따름.
           
            outHeight = inHeight;
            outWidth = inWidth;
            // 출력 2차원 배열을 준비
            outImageArray = new Array(outHeight); // 256짜리 1차원 배열
            for(var i=0; i<outHeight; i++) 
                outImageArray[i] = new Array(outWidth);

            // ***** 진짜 영상처리 알고리즘 *****
            var value = parseInt(prompt("밝게할 값", "0"));
            for(var i=0; i<inHeight; i++) {
                for (var k=0; k<inWidth; k++) {
                    // 문자 --> 숫자
                    pixel = inImageArray[i][k].charCodeAt(0);
                    // **** 요기가 핵심 알고리즘. (밝게하기)
                    if (pixel + value > 255)
                        pixel = 255;
                    else 
                        pixel += value;
                    // 숫자 --> 문자
                    outImageArray[i][k] = String.fromCharCode(pixel);
                }
            }
          printOutImage();
        }
                function  negativeImage() {  // 흑백 알고리즘
            // (중요!) 출력 영상의 크기를 결정... 알고리즘에 따름.
            outHeight = inHeight;
            outWidth = inWidth;
            // 출력 2차원 배열을 준비
            outImageArray = new Array(outHeight); // 256짜리 1차원 배열
            for(var i=0; i<outHeight; i++) 
                outImageArray[i] = new Array(outWidth);
          
            for(var i=0; i<inHeight; i++) {
                for (var k=0; k<inWidth; k++) {
                    // 문자 --> 숫자
                    pixel = inImageArray[i][k].charCodeAt(0);
                    pixel = 255 - pixel;
                    outImageArray[i][k] = String.fromCharCode(pixel);
                }
            }
          printOutImage();
        }

      

        function addDark() {
            outHeight = inHeight;
            outWidth = inWidth;

            outImageArray = new Array(outHeight); 
            for(var i=0; i<inHeight; i++) {
                outImageArray[i] = new Array(outWidth);
                for (var k=0; k<inWidth; k++) {
                    // 문자 --> 숫자
                    pixel = inImageArray[i][k].charCodeAt(0);
                    // ** 요기가 핵심 알고리즘. (밝게하기)
                    pixel -= 50;
                    // 숫자 --> 문자
                    outImageArray[i][k] = String.fromCharCode(pixel);
                }
            
                }
                printOutImage();
            }


        function grayscale() {
            outHeight = inHeight;
            outWidth = inWidth;

            outImageArray = new Array(outHeight); 

            for(var i=0; i<inHeight; i++) {
                outImageArray[i] = new Array(outWidth);
                for (var k=0; k<inWidth; k++) {
         
                       pixel = inImageArray[i][k].charCodeAt(0);
                    if(pixel > 127)
                         pixel=255;
                    else
                         pixel = 0;

                        outImageArray[i][k] = String.fromCharCode(pixel);
                }
            }
            printOutImage();
                  }
        
            function leftmirr(){
                outHeight = inHeight;
                outWidth = inWidth;

                outImageArray = new Array(outHeight); 
        
                for(var i=0; i<inHeight; i++) {
                    outImageArray[i] = new Array(outWidth);
                for (var k=0; k<outWidth; k++) {
                    pixel = inImageArray[i][outWidth-k-1].charCodeAt(0);

                    outImageArray[i][k] = String.fromCharCode(pixel);

                  }
               }
        printOutImage();
            }    

            function upmirr(){
                outHeight = inHeight;
                outWidth = inWidth;

                outImageArray = new Array(outHeight); 

                for(var i=0; i<inHeight; i++) {
                    outImageArray[i] = new Array(outWidth);
                for (var k=0; k<outWidth; k++) {
                    pixel= inImageArray[outHeight-i-1][k].charCodeAt(0);
                   
                    outImageArray[i][k] = String.fromCharCode(pixel);
                }
        }
        printOutImage();
            }    


    </script>

</head>
<body onload='init()'>
    <input type='file' id='selectFile' onchange='readRawImage()'/>
    <br>
    <input type='button' id='photoAdd' value='밝게하기' onclick='addImage()'/>
    <input type='button' id='photoNegative' value='반전변화' onclick='negativeImage()'/>
    <input type='button' id='photoSub' value='어둡게하기' onclick='addDark()'/>
    <input type='button' id='photoSub' value='흑백처리' onclick='grayscale()'/>
    <input type='button' id='photoSub' value='미러링(좌우)' onclick='leftmirr()'/>
    <input type='button' id='photoSub' value='미러링(상하)' onclick='upmirr()'/>
   
    <br>
    <canvas id='inCanvas' style='background-color:rgb(248, 209, 164)'></canvas>
    <canvas id='outCanvas' style='background-color:rgb(164, 248, 192)'></canvas>
</body>
</html>
