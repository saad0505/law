<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <title>مقابلة قانونية</title>
  <!-- html2pdf.js لتحويل HTML إلى PDF مع دعم UTF-8 -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
  <!-- خط Cairo للنص العربي -->
  <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;700&display=swap" rel="stylesheet">
  <style>
    :root {
      --primary: #305496;
      --secondary: #567ace;
      --bg: #f8f9fa;
      --text: #333;
    }
    body {
      font-family: 'Cairo', sans-serif;
      background: var(--bg);
      direction: rtl;
      margin: 0;
      padding: 0;
    }
    .container {
      max-width: 800px;
      width: 90%;
      margin: 40px auto;
      background: #fff;
      padding: 30px;
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }
    h3 {
      color: var(--primary);
      text-align: center;
      margin-bottom: 24px;
      font-size: 22px;
      font-weight: 700;
    }
    label {
      display: block;
      font-weight: 600;
      margin: 16px 0 8px;
      color: var(--text);
    }
    input, textarea {
      width: 100%;
      padding: 10px 12px;
      border: 1px solid #ccd0d5;
      border-radius: 4px;
      font-size: 14px;
      color: var(--text);
      box-sizing: border-box;
    }
    textarea { resize: vertical; min-height: 80px; }
    button {
      background: var(--primary);
      color: #fff;
      border: none;
      border-radius: 4px;
      padding: 12px 24px;
      font-size: 15px;
      font-weight: 600;
      cursor: pointer;
      margin: 24px auto 0;
      display: block;
      transition: background 0.3s;
    }
    button:hover { background: var(--secondary); }
    /* PDF content */
    #pdfContent {
      display: none;
      width: 210mm;
      min-height: 297mm;
      padding: 20mm;
      margin: 0;
      background: #fff;
      box-sizing: border-box;
    }
    #pdfContent .pdf-header {
      width: 100%;
      padding-bottom: 8pt;
      margin-bottom: 12pt;
      border-bottom: 2pt solid var(--primary);
      font-size: 24pt;
      font-weight: 700;
      color: var(--primary);
      text-align: center;
    }
    #pdfContent .user-info {
      display: flex;
      justify-content: space-between;
      background: #eef2fb;
      padding: 8pt;
      border-radius: 4pt;
      margin-bottom: 16pt;
      font-size: 11pt;
      color: var(--text);
    }
    #pdfContent .user-info div { width: 32%; }
    #pdfContent .qa-card {
      margin-bottom: 14pt;
      padding: 10pt;
      border: 1pt solid #dde0e6;
      border-left: 6pt solid var(--primary);
      border-radius: 4pt;
      page-break-inside: avoid;
    }
    #pdfContent .qa-card .question {
      font-size: 12pt;
      font-weight: 600;
      margin-bottom: 4pt;
      color: var(--primary);
    }
    #pdfContent .qa-card .answer {
      font-size: 11pt;
      line-height: 1.5;
      color: var(--text);
      white-space: pre-wrap;
    }
  </style>
</head>
<body>
  <div class="container" id="userForm">
    <h3>البيانات الشخصية</h3>
    <label for="fullName">الاسم الكامل</label>
    <input type="text" id="fullName" placeholder="الاسم الكامل" required>
    <label for="phone">رقم الجوال</label>
    <input type="text" id="phone" maxlength="10" placeholder="05XXXXXXXX" oninput="this.value=this.value.replace(/[^0-9]/g,'')" required>
    <label for="expectedSalary">الراتب المتوقع</label>
    <input type="text" id="expectedSalary" maxlength="5" placeholder="الراتب المتوقع" oninput="this.value=this.value.replace(/[^0-9]/g,'')" required>
    <button onclick="startInterview()">بدء المقابلة</button>
  </div>

  <div class="container" id="interview" style="display:none;">
    <h3>مقابلة قانونية</h3>
    <div id="questions"></div>
    <button onclick="generatePdf()">تحميل الإجابات PDF</button>
  </div>

  <div class="container" id="thanks" style="display:none; text-align:center;">
    <h3>شكرًا لمشاركتك!</h3>
    <p>تم تحميل الإجابات بنجاح.</p>
  </div>

  <!-- PDF content hidden -->
  <div id="pdfContent">
    <div class="pdf-header">إجابات المقابلة القانونية</div>
    <div class="user-info">
      <div>الاسم: <strong id="pdfName"></strong></div>
      <div>الجوال: <strong id="pdfPhone"></strong></div>
      <div>الراتب المتوقع: <strong id="pdfSalary"></strong></div>
    </div>
    <div id="pdfQuestions"></div>
  </div>

  <script>
    let userName, userPhone, userSalary;
    const questions = [
      "1- ما هي آلية إدارة شركة ذات مسؤولية محدودة متعددة الشركاء والمدراء إذا لم ينص عقد التأسيس على ذلك، ولا يوجد مجلس مدراء، مع تساوي نسب الشركاء وعدم إمكانية التوافق بينهم؟",
      "2- هل أجاز النظام لمدير الشركة تحرير سند لأمر أو شيك لنفسه؟ وما هو المسوغ النظامي لذلك؟",
      "3- هل يجوز لأحد الشركاء، إن كان مديناً لشركة هو شريك بها، أن يتقدم لقاضي التنفيذ بالحجز على حصصه في الشركة للوفاء بالدين الذي في ذمته للشركة؟"
    ];
    function startInterview() {
      userName = document.getElementById('fullName').value.trim();
      userPhone = document.getElementById('phone').value.trim();
      userSalary = document.getElementById('expectedSalary').value.trim();
      if (!userName||!userPhone||!userSalary) { alert('يرجى تعبئة جميع الحقول.'); return; }
      if (!/^05\d{8}$/.test(userPhone)) { alert('رقم الجوال غير صالح.'); return; }
      if (!/^\d{1,5}$/.test(userSalary)) { alert('الراتب المتوقع غير صالح.'); return; }
      document.getElementById('userForm').style.display='none';
      document.getElementById('interview').style.display='block';
      const qDiv=document.getElementById('questions'); qDiv.innerHTML='';
      questions.forEach((q,i)=>{
        qDiv.innerHTML+=`<label>${q}</label><textarea id='q${i}' rows='4'></textarea>`;
      });
    }
    function generatePdf() {
      document.getElementById('pdfName').textContent=userName;
      document.getElementById('pdfPhone').textContent=userPhone;
      document.getElementById('pdfSalary').textContent=userSalary;
      let html='';
      questions.forEach((q,i)=>{
        const ans=document.getElementById(`q${i}`).value.trim()||'-';
        html+=`<div class='qa-card'><div class='question'>${q}</div><div class='answer'>${ans}</div></div>`;
      });
      document.getElementById('pdfQuestions').innerHTML=html;
      const el=document.getElementById('pdfContent'); el.style.display='block';
      html2pdf().set({
        margin: [10,10,10,10],
        filename:'الإجابات.pdf',
        jsPDF:{unit:'mm',format:'a4',orientation:'portrait'},
        html2canvas:{scale:2,useCORS:true},
        pagebreak:{mode:['avoid-all','css','legacy']}
      }).from(el).save().then(()=>{
        el.style.display='none'; document.getElementById('interview').style.display='none'; document.getElementById('thanks').style.display='block';
      });
    }
  </script>
</body>
</html>
