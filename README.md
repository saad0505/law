<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <title>مقابلة قانونية</title>
  <!-- مكتبة html2pdf لتحويل HTML إلى PDF مع دعم UTF-8 -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
  <!-- خط Cairo لتحسين عرض النصوص العربية -->
  <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;700&display=swap" rel="stylesheet">
  <style>
    body { font-family: 'Cairo', sans-serif; background-color: #f3f3f3; direction: rtl; margin: 0; padding: 0; }
    .container { max-width: 800px; width: 90%; margin: 30px auto; background: #fff; padding: 30px; border-radius: 12px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
    h3 { color: #2b579a; margin-bottom: 20px; text-align: center; }
    label { font-weight: bold; display: block; margin: 15px 0 5px; }
    input, textarea { width: 100%; padding: 10px; margin-bottom: 15px; border: 1px solid #ccc; border-radius: 6px; box-sizing: border-box; }
    button { display: inline-block; background: #0078d4; color: #fff; border: none; border-radius: 6px; padding: 10px 20px; cursor: pointer; transition: background 0.2s; margin-top: 10px; }
    button:hover { background: #005a9e; }
    /* محتوى PDF مخفي إفتراضياً */
    #pdfContent { display: none; font-family: 'Cairo', sans-serif; background: #ffffff; padding: 24pt; box-sizing: border-box; width: 595pt; margin: 0 auto; }
    #pdfContent .pdf-header { background: #2b579a; color: #ffffff; padding: 12pt; border-radius: 6pt 6pt 0 0; text-align: center; font-size: 18pt; margin-bottom: 20pt; }
    #pdfContent .user-info { display: flex; justify-content: space-between; background: #f0f8ff; padding: 12pt; border-radius: 6pt; margin-bottom: 20pt; font-size: 11pt; }
    #pdfContent .user-info div { width: 32%; }
    #pdfContent .qa-card { border: 1pt solid #ddd; border-radius: 6pt; padding: 10pt; margin-bottom: 12pt; }
    #pdfContent .qa-card .question { color: #005a9e; font-size: 13pt; font-weight: bold; margin-bottom: 6pt; }
    #pdfContent .qa-card .answer { color: #333; font-size: 12pt; background: #fafafa; padding: 8pt; border-radius: 4pt; line-height: 1.4; }
  </style>
</head>
<body>
  <div class="container">
    <!-- شاشة بيانات المستخدم -->
    <div id="userForm">
      <h3>البيانات الشخصية</h3>
      <label for="fullName">الاسم الكامل:</label>
      <input type="text" id="fullName" placeholder="الاسم الكامل" required>
      <label for="phone">رقم الجوال:</label>
      <input type="text" id="phone" inputmode="numeric" maxlength="10" placeholder="05XXXXXXXX" oninput="this.value=this.value.replace(/[^0-9]/g,'')" required>
      <label for="expectedSalary">الراتب المتوقع:</label>
      <input type="text" id="expectedSalary" inputmode="numeric" maxlength="5" placeholder="الراتب المتوقع" oninput="this.value=this.value.replace(/[^0-9]/g,'')" required>
      <button onclick="validateAndStart()">بدء المقابلة</button>
    </div>

    <!-- شاشة المقابلة -->
    <div id="interview" style="display:none;">
      <h3>مقابلة قانونية</h3>
      <div id="questions"></div>
      <button onclick="preparePdf()">تحميل الإجابات PDF</button>
    </div>

    <!-- شاشة الشكر -->
    <div id="thanks" style="display:none; text-align:center;">
      <h3>شكرًا لمشاركتك!</h3>
      <p>تم تحميل إجاباتك بنجاح.</p>
    </div>
  </div>

  <!-- محتوى PDF مخفي -->
  <div id="pdfContent">
    <div class="pdf-header">مقابلة قانونية - إجابات المتقدم</div>
    <div class="user-info">
      <div>الاسم: <strong id="pdfName"></strong></div>
      <div>الجوال: <strong id="pdfPhone"></strong></div>
      <div>الراتب المتوقع: <strong id="pdfSalary"></strong></div>
    </div>
    <div id="pdfQuestions"></div>
  </div>

  <script>
    let userName = '', userPhone = '', userSalary = '';
    const questions = [
      "1- ما هي آلية إدارة شركة ذات مسؤولية محدودة متعددة الشركاء والمدراء إذا لم ينص عقد التأسيس على ذلك، ولا يوجد مجلس مدراء، مع تساوي نسب الشركاء وعدم إمكانية التوافق بينهم؟",
      "2- هل أجاز النظام لمدير الشركة تحرير سند لأمر أو شيك لنفسه؟ وما هو المسوغ النظامي لذلك؟",
      "3- هل يجوز لأحد الشركاء، إن كان مديناً لشركة هو شريك بها، أن يتقدم لقاضي التنفيذ بالحجز على حصصه في الشركة للوفاء بالدين الذي في ذمته للشركة؟"
    ];

    function validateAndStart() {
      userName = document.getElementById('fullName').value.trim();
      userPhone = document.getElementById('phone').value.trim();
      userSalary = document.getElementById('expectedSalary').value.trim();
      if (!userName || !userPhone || !userSalary) { alert('يرجى تعبئة جميع الحقول.'); return; }
      if (!/^05\d{8}$/.test(userPhone)) { alert('رقم الجوال يجب أن يتكون من 10 أرقام ويبدأ بـ05.'); return; }
      if (!/^\d{1,5}$/.test(userSalary)) { alert('الراتب المتوقع يجب أن يكون رقماً ولا يزيد عن 5 أرقام.'); return; }
      document.getElementById('userForm').style.display = 'none';
      startInterview();
    }

    function startInterview() {
      let qDiv = '';
      questions.forEach((q, i) => {
        qDiv += `<label>${q}</label><textarea id="q${i}" rows="4"></textarea>`;
      });
      document.getElementById('questions').innerHTML = qDiv;
      document.getElementById('interview').style.display = 'block';
    }

    function preparePdf() {
      document.getElementById('pdfName').textContent = userName;
      document.getElementById('pdfPhone').textContent = userPhone;
      document.getElementById('pdfSalary').textContent = userSalary;
      let html = '';
      questions.forEach((q, i) => {
        const ans = document.getElementById(`q${i}`).value || '-';
        html += `<div class="qa-card"><div class="question">${q}</div><div class="answer">${ans}</div></div>`;
      });
      document.getElementById('pdfQuestions').innerHTML = html;
      const element = document.getElementById('pdfContent');
      element.style.display = 'block';
      const opt = { margin: 10, filename: 'الإجابات.pdf', image: { type: 'jpeg', quality: 0.98 }, html2canvas: { scale: 2, useCORS: true }, jsPDF: { unit: 'pt', format: 'a4', orientation: 'portrait' }, pagebreak: { mode: ['avoid-all', 'css', 'legacy'] } };
      html2pdf().set(opt).from(element).save().then(() => {
        element.style.display = 'none';
        document.getElementById('interview').style.display = 'none';
        document.getElementById('thanks').style.display = 'block';
      });
    }
  </script>
</body>
</html>
