const airtableToken = "YOUR_AIRTABLE_TOKEN";
const baseId = "YOUR_BASE_ID";
const table = "Tracking";

async function fetchTrackingBatch(numbers) {
  if (!numbers.length) return [];
  const formula = `OR(${numbers.map(n => `{TrackingNumber}='${n}'`).join(",")})`;
  const url = `https://api.airtable.com/v0/${baseId}/${table}?filterByFormula=${encodeURIComponent(formula)}&sort[0][field]=DateTime&sort[0][direction]=asc`;
  const res = await fetch(url, { headers: { Authorization: `Bearer ${airtableToken}` } });
  const data = await res.json();
  return data.records.map(r => r.fields);
}

function groupBy(numbersList, records){
  const map = {};
  numbersList.forEach(n => map[n] = []);
  records.forEach(f => {
    if (map[f.TrackingNumber]) map[f.TrackingNumber].push(f);
  });
  return map;
}

async function track(){
  const txt = document.getElementById("trackingNumbers").value.trim();
  const nums = Array.from(new Set(txt.split(/[\s,]+/).filter(n => n)));
  const resDiv = document.getElementById("result");
  resDiv.innerHTML = `<p>⏳ กำลังโหลด...</p>`;
  try {
    const records = await fetchTrackingBatch(nums);
    const grouped = groupBy(nums, records);
    let html = "";
    nums.forEach(n => {
      html += `<div class="track-item"><h3>${n}</h3>`;
      const items = grouped[n];
      if (items.length) {
        html += "<ul>";
        items.forEach(i => html += `<li>${i.DateTime} — ${i.Status} (${i.Courier})</li>`);
        html += "</ul>";
      } else {
        html += `<p style="color:red;">ไม่พบข้อมูล</p>`;
      }
      html += "</div>";
    });
    resDiv.innerHTML = html;
  } catch(e) {
    resDiv.innerHTML = `<p style="color:red;">❌ เกิดข้อผิดพลาด</p>`;
  }
}
