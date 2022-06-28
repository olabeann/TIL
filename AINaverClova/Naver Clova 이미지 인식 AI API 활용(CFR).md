<h3> Naver Clova 이미지 인식 AI API 활용(CFR)</h3>

- Clova Face Recognition API(=CFR API)는 이미지 데이터를 입력 받은 후 얼굴 인식한 결과를 JSON 형태로 

  반환 한다.

- CFR API는 이미지에 있는 얼굴을 인식해서 분석 정보를 제공하는 얼굴 감지 API와 닮은 연예인을

​       알려주는 유명인 얼굴 인식 API를 제공한다.

* HTTP 기반의 REST API이며, 로그인이 필요하지 않은 비로그인 Open API이다.
* 공식 문서 : https://developers.naver.com/docs/clova/api/CFR/API_Guide.md#%EC%9D%91%EB%8B%B5-2  

🎇국내 유명인 위주로 인식을 잘함



1. https://www.ncloud.com/ 로그인해서 CFR을 이용 신청 + Application을 등록 해야 한다.

2. Application을 들어가서 인증 정보에 ID 값과 클라이언트 시크릿값을 복사해서 메모 한다.

3. Java 코드(clientId랑 clientSecret 복사해서 붙여넣고, imgFile = "" 사용할 이미지의 파일 경로를 넣는다. )

   ```java
   import java.io.*;
   import java.net.HttpURLConnection;
   import java.net.URL;
   import java.net.URLConnection;
    
   // 네이버 얼굴인식 API 예제
   public class APIExamFace {
    
       public static void main(String[] args) {
    
           StringBuffer reqStr = new StringBuffer();
           String clientId = "YOUR_CLIENT_ID";//애플리케이션 클라이언트 아이디값";
           String clientSecret = "YOUR_CLIENT_SECRET";//애플리케이션 클라이언트 시크릿값";
    
           try {
               String paramName = "image"; // 파라미터명은 image로 지정
               String imgFile = "이미지 파일 경로 ";
               File uploadFile = new File(imgFile);
               String apiURL = "https://naveropenapi.apigw.ntruss.com/vision/v1/celebrity"; // 유명인 얼굴 인식
               //String apiURL = "https://naveropenapi.apigw.ntruss.com/vision/v1/face"; // 얼굴 감지
               URL url = new URL(apiURL);
               HttpURLConnection con = (HttpURLConnection)url.openConnection();
               con.setUseCaches(false);
               con.setDoOutput(true);
               con.setDoInput(true);
               // multipart request
               String boundary = "---" + System.currentTimeMillis() + "---";
               con.setRequestProperty("Content-Type", "multipart/form-data; boundary=" + boundary);
               con.setRequestProperty("X-NCP-APIGW-API-KEY-ID", clientId);
               con.setRequestProperty("X-NCP-APIGW-API-KEY", clientSecret);
               OutputStream outputStream = con.getOutputStream();
               PrintWriter writer = new PrintWriter(new OutputStreamWriter(outputStream, "UTF-8"), true);
               String LINE_FEED = "\r\n";
               // file 추가
               String fileName = uploadFile.getName();
               writer.append("--" + boundary).append(LINE_FEED);
               writer.append("Content-Disposition: form-data; name=\"" + paramName + "\"; filename=\"" + fileName + "\"").append(LINE_FEED);
               writer.append("Content-Type: "  + URLConnection.guessContentTypeFromName(fileName)).append(LINE_FEED);
               writer.append(LINE_FEED);
               writer.flush();
               FileInputStream inputStream = new FileInputStream(uploadFile);
               byte[] buffer = new byte[4096];
               int bytesRead = -1;
               while ((bytesRead = inputStream.read(buffer)) != -1) {
                   outputStream.write(buffer, 0, bytesRead);
               }
               outputStream.flush();
               inputStream.close();
               writer.append(LINE_FEED).flush();
               writer.append("--" + boundary + "--").append(LINE_FEED);
               writer.close();
               BufferedReader br = null;
               int responseCode = con.getResponseCode();
               if(responseCode==200) { // 정상 호출
                   br = new BufferedReader(new InputStreamReader(con.getInputStream()));
               } else {  // 오류 발생
                   System.out.println("error!!!!!!! responseCode= " + responseCode);
                   br = new BufferedReader(new InputStreamReader(con.getInputStream()));
               }
               String inputLine;
               if(br != null) {
                   StringBuffer response = new StringBuffer();
                   while ((inputLine = br.readLine()) != null) {
                       response.append(inputLine);
                   }
                   br.close();
                   //System.out.println(response.toString()); 
                   
                   //String to Json Object
                   JSONObject jsonObj = new JSONObject(response.toString());
                   JSONObject jsonObj2 = (JSONObject) jsonObj.get("info"); //https://jsonbeautifier.org/ 에서 확인
                   int  facecnt = (int)jsonObj2.get("faceCount");
                   System.out.println("발견된 얼굴 수 :"+facecnt);
                   
                   JSONArray jsonArr = (JSONArray)jsonObj.get("faces");
                   
                  // System.out.println(jsonArr); //https://jsonbeautifier.org/ 에서 확인
    
                   for(int i= 0 ; i<jsonArr.length(); i++) {
                     String name  = jsonArr.getJSONObject(i).getJSONObject("celebrity").get("value").toString();
                     String per = jsonArr.getJSONObject(i).getJSONObject("celebrity").get("confidence").toString();
                     double per2 = Double.parseDouble(per);
                     System.out.printf("감지된 얼굴: %d : %s %7.2f %% \n",(i+1),name,(per2*100));  
                     
                  }
    
               } else {
                   System.out.println("error !!!");
               }
           } catch (Exception e) {
               System.out.println(e);
           }
       }
   }
   



💎 결과 창 : 나는 허영지를 55% 닮았다......![네이버 CFR 콘솔](https://user-images.githubusercontent.com/103403634/176204859-7cdaa7e5-92f7-42e0-9795-8a48a9a491c4.PNG)