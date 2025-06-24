<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <title>مقابلة قانونية - تسجيل الدخول</title>
  <script src="https://cdn.emailjs.com/sdk/latest/email.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    body { font-family: 'Segoe UI', sans-serif; background-color: #f3f3f3; direction: rtl; }
    .container { max-width: 800px; margin:50px auto; background:#fff; padding:40px; border-radius:12px; box-shadow:0 2px 12px rgba(0,0,0,0.1); }
    button { padding:10px 20px; cursor:pointer; }
  </style>
</head>
<body>
<div class="container">
  <div id="login">
    <h3>تسجيل الدخول</h3>
    <input type="email" id="emailLogin" placeholder="البريد الإلكتروني"><br><br>
    <input type="text" id="codeLogin" placeholder="كود الدخول"><br><br>
    <button onclick="login()">دخول</button>
  </div>

  <div id="interview" style="display:none;">
    <h3>مقابلة قانونية</h3>
    <div id="questions"></div>
    <button onclick="submitInterview()">إرسال الإجابات</button>
  </div>

  <div id="thanks" style="display:none;">
    <h3>شكرًا لمشاركتك!</h3>
  </div>
</div>

<script>
emailjs.init('YOUR_EMAILJS_PUBLIC_KEY'); // ضع مفتاحك هنا

let allowedCodes = JSON.parse(localStorage.getItem('allowedCodes')) || Array.from({length:26},(_,i)=>`ACC${String(i+1).padStart(2,'0')}`);
let usedCodes = JSON.parse(localStorage.getItem('usedCodes')) || [];

function login(){
  const code=document.getElementById('codeLogin').value.trim();
  if(usedCodes.includes(code)){alert('تم استخدام هذا الكود مسبقًا.');return;}
  if(allowedCodes.includes(code)){
    usedCodes.push(code);localStorage.setItem('usedCodes',JSON.stringify(usedCodes));
    document.getElementById('login').style.display='none';
    startInterview();
  }else{alert('كود الدخول غير صحيح.');}
}

const questions=[
  "1- آلية إدارة شركة ذات مسؤولية محدودة متعددة الشركاء والمدراء إذا لم ينص عقد التأسيس على ذلك، ولا يوجد مجلس مدراء؟",
  "2- هل أجاز النظام لمدير الشركة تحرير سند لأمر أو شيك لنفسه؟ وما هو المسوغ؟",
  "3- هل يجوز لأحد الشركاء المدين لشركته الحجز على حصصه للوفاء بالدين؟"
];

function startInterview(){
  let qDiv='';
  questions.forEach((q,i)=>{
    qDiv+=`<p>${q}</p><textarea id="q${i}" rows="4" style="width:100%;"></textarea>`;
  });
  document.getElementById('questions').innerHTML=qDiv;
  document.getElementById('interview').style.display='block';
}

function submitInterview(){
  const { jsPDF }=window.jspdf;
  const doc=new jsPDF();
  let y=10;
  questions.forEach((q,i)=>{
    doc.text(q,10,y); y+=10;
    const ans=document.getElementById(`q${i}`).value;
    doc.text(ans,10,y); y+=20;
  });
  const pdfBase64=doc.output('datauristring');

  emailjs.send('YOUR_SERVICE_ID','YOUR_TEMPLATE_ID',{
    to_email:'saad1alhumaidi@gmail.com',
    message:'مرفق إجابات المقابلة القانونية',
    attachment:pdfBase64
  }).then(()=>{
    alert('تم إرسال الإجابات بنجاح!');
    document.getElementById('interview').style.display='none';
    document.getElementById('thanks').style.display='block';
  }).catch(()=>alert('حدث خطأ في إرسال البريد.'));
}
</script>
</body>
</html>
