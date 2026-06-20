# 📊 구글 스프레드시트 연동 가이드 (Google Apps Script)

자리 배치 결과를 구글 시트에 자동으로 저장하기 위한 구글 앱스 스크립트(GAS) 설정법입니다. 
아래 순서대로 따라 하시면 5분 안에 연동을 마칠 수 있습니다! ✨

---

## 1단계. 구글 스프레드시트 생성 및 스크립트 실행
1. [구글 스프레드시트](https://sheets.google.com)로 이동하여 새 스프레드시트를 생성합니다.
2. 스프레드시트 상단 메뉴에서 **[확장 프로그램] ➡ [Apps Script]**를 클릭합니다.

---

## 2단계. 스크립트 코드 복사 및 붙여넣기
열린 Apps Script 에디터 창의 기존 코드를 모두 지우고, **아래 코드 전체를 복사하여 붙여넣습니다.**

```javascript
function doPost(e) {
  try {
    var json = JSON.parse(e.postData.contents);
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    
    var timestamp = new Date();
    var formattedDate = Utilities.formatDate(timestamp, Session.getScriptTimeZone(), "yyyy-MM-dd HH:mm:ss");
    
    var rows = json.rows;
    var cols = json.cols;
    var seating = json.seating; // 학생 이름 배열

    // 1. 저장 구분선 기록
    sheet.appendRow(["💖 자리 배치 기록 (" + formattedDate + ")", "규격: " + rows + "행 x " + cols + "열"]);
    
    // 2. 칠판 가이드 기록 (구글 시트 상단 방향)
    var headerGuide = [];
    for (var c = 0; c < cols; c++) {
      headerGuide.push(c === 0 ? "👩‍🏫 [선생님 교단(칠판)]" : "");
    }
    sheet.appendRow(headerGuide);

    // 3. 자리 배치 구조 그대로 구글 시트 행에 기록
    for (var r = 0; r < rows; r++) {
      var rowData = [];
      for (var c = 0; c < cols; c++) {
        var idx = r * cols + c;
        var name = seating[idx] || "💤 빈자리";
        rowData.push(name);
      }
      sheet.appendRow(rowData);
    }
    
    // 4. 구분용 빈 칸 삽입
    sheet.appendRow([]); 

    // 성공 응답 반환
    return ContentService.createTextOutput(JSON.stringify({ result: 'success' }))
      .setMimeType(ContentService.MimeType.JSON)
      .setHeader('Access-Control-Allow-Origin', '*'); // CORS 에러 방지
      
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({ result: 'error', message: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON)
      .setHeader('Access-Control-Allow-Origin', '*');
  }
}
```

- 붙여넣은 뒤 상단의 **[저장 (💾 모양 버튼)]**을 누릅니다.

---

## 3단계. 웹앱 배포 (가장 중요! ⭐)
1. Apps Script 창 우측 상단의 **[배포] ➡ [새 배포]**를 선택합니다.
2. 유형 선택(톱니바퀴 아이콘)에서 **[웹앱]**을 클릭합니다.
3. 배포 설정을 다음과 같이 지정합니다:
   - **설명**: `자리배치 연동` (아무거나 적으셔도 됩니다)
   - **다음 사용자로 실행**: **[나(본인 구글 이메일)]**
   - **액세스 권한이 있는 사용자**: **[모든 사람(Anyone)]** 
     *(⭐ 주의: '모든 사람'으로 설정해야 웹앱에서 구글 로그인을 거치지 않고 저장할 수 있습니다.)*
4. **[배포]** 버튼을 누릅니다.
5. 최초 배포 시 구글 계정 액세스 승인창이 뜹니다. **[권한 검토]**를 누르고 본인 구글 계정을 선택한 후, 경고 화면이 나오면 좌측 하단의 **Advanced(고급)**을 누르고 **Go to Untitled project (unsafe)** 혹은 **프로젝트로 이동**을 클릭하여 허용해 줍니다.
6. 배포가 완료되면 화면에 생성되는 **웹앱 URL**을 복사합니다.
   *(예시 주소 형태: `https://script.google.com/macros/s/.../exec`)*

---

## 4단계. 웹앱에 주소 입력하고 전송하기
1. 학급 자리 배치 웹앱([index.html](index.html))을 실행합니다.
2. 좌측 제어판 하단에 있는 **구글 시트 연동 설정**란에 방금 복사한 **웹앱 URL**을 붙여넣습니다.
3. 자리를 배치한 후, 제어판에서 **📊 구글 시트에 저장하기** 버튼을 누르면 스프레드시트에 실시간으로 배치가 시각화되어 저장됩니다!
