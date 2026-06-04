export const config = { api: { bodyParser: false } };

export default async function handler(req, res) {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, x-api-key');

  if (req.method === 'OPTIONS') return res.status(200).end();
  if (req.method !== 'POST') return res.status(405).json({ error: 'Method not allowed' });

  try {
    const chunks = [];
    for await (const chunk of req) chunks.push(chunk);
    const buffer = Buffer.concat(chunks);
    const boundary = req.headers['content-type'].split('boundary=')[1];

    const parts = parseMultipart(buffer, boundary);
    const apiKey = parts.apiKey;
    const transcript = parts.transcript || '';
    const dimensions = parts.dimensions ? JSON.parse(parts.dimensions) : [];
    const videoBuffer = parts.video;
    const videoMime = parts.videoMime || 'video/mp4';
    const videoName = parts.videoName || 'video.mp4';

    if (!apiKey) return res.status(400).json({ error: '缺少 API Key' });
    if (!videoBuffer) return res.status(400).json({ error: '缺少视频文件' });

    // Step 1: Upload video to Gemini
    const uploadRes = await fetch(
      'https://generativelanguage.googleapis.com/upload/v1beta/files?uploadType=multipart',
      {
        method: 'POST',
        headers: { 'X-Goog-Api-Key': apiKey },
        body: buildMultipart(videoBuffer, videoMime, videoName),
      }
    );

    if (!uploadRes.ok) {
      const err = await uploadRes.json();
      return res.status(400).json({ error: err.error?.message || '视频上传失败' });
    }

    const uploadData = await uploadRes.json();
    const fileUri = uploadData.file?.uri;
    const fileName = uploadData.file?.name;

    // Step 2: Wait for video to be processed
    let state = 'PROCESSING';
    let attempts = 0;
    while (state === 'PROCESSING' && attempts < 20) {
      await sleep(3000);
      const checkRes = await fetch(
        `https://generativelanguage.googleapis.com/v1beta/${fileName}`,
        { headers: { 'X-Goog-Api-Key': apiKey } }
      );
      const checkData = await checkRes.json();
      state = checkData.state;
      attempts++;
    }

    if (state !== 'ACTIVE') {
      return res.status(400).json({ error: '视频处理超时，请重试' });
    }

    // Step 3: Analyze with Gemini
    const transcriptText = transcript ? `\n\n字幕/文字稿：\n${transcript}` : '';
    const prompt = `你是一位顶级短视频爆款分析师，请对这个视频进行深度拆解。

请分析以下维度：
${dimensions.map((d, i) => `${i + 1}. ${d}`).join('\n')}
${transcriptText}

请用中文回答，每个维度单独输出，格式：
【维度名称】
• 分析点1
• 分析点2
• 分析点3

要求：直接、犀利、可操作，像在给创作者做实战复盘。`;

    const genRes = await fetch(
      `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`,
      {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contents: [{
            parts: [
              { fileData: { mimeType: videoMime, fileUri } },
              { text: prompt }
            ]
          }]
        })
      }
    );

    if (!genRes.ok) {
      const err = await genRes.json();
      return res.status(400).json({ error: err.error?.message || '分析失败' });
    }

    const genData = await genRes.json();
    const text = genData.candidates?.[0]?.content?.parts?.[0]?.text;
    if (!text) return res.status(400).json({ error: '未获取到分析结果' });

    return res.status(200).json({ result: text });

  } catch (err) {
    return res.status(500).json({ error: err.message });
  }
}

function sleep(ms) { return new Promise(r => setTimeout(r, ms)); }

function parseMultipart(buffer, boundary) {
  const parts = {};
  const boundaryBuf = Buffer.from('--' + boundary);
  let start = 0;

  while (start < buffer.length) {
    const bStart = bufferIndexOf(buffer, boundaryBuf, start);
    if (bStart === -1) break;
    const headerStart = bStart + boundaryBuf.length + 2;
    const headerEnd = bufferIndexOf(buffer, Buffer.from('\r\n\r\n'), headerStart);
    if (headerEnd === -1) break;
    const headerStr = buffer.slice(headerStart, headerEnd).toString();
    const nextBoundary = bufferIndexOf(buffer, boundaryBuf, headerEnd);
    const bodyEnd = nextBoundary === -1 ? buffer.length : nextBoundary - 2;
    const body = buffer.slice(headerEnd + 4, bodyEnd);

    const nameMatch = headerStr.match(/name="([^"]+)"/);
    const filenameMatch = headerStr.match(/filename="([^"]+)"/);
    const mimeMatch = headerStr.match(/Content-Type: ([^\r\n]+)/);

    if (nameMatch) {
      const name = nameMatch[1];
      if (filenameMatch) {
        parts[name] = body;
        parts[name + 'Mime'] = mimeMatch ? mimeMatch[1].trim() : 'application/octet-stream';
        parts[name + 'Name'] = filenameMatch[1];
      } else {
        parts[name] = body.toString().trim();
      }
    }
    start = nextBoundary === -1 ? buffer.length : nextBoundary;
  }
  return parts;
}

function bufferIndexOf(buf, search, start = 0) {
  for (let i = start; i <= buf.length - search.length; i++) {
    let found = true;
    for (let j = 0; j < search.length; j++) {
      if (buf[i + j] !== search[j]) { found = false; break; }
    }
    if (found) return i;
  }
  return -1;
}

function buildMultipart(videoBuffer, mimeType, filename) {
  const boundary = 'gem_boundary_' + Date.now();
  const meta = JSON.stringify({ file: { display_name: filename } });
  const header = Buffer.from(
    `--${boundary}\r\nContent-Type: application/json; charset=utf-8\r\n\r\n${meta}\r\n--${boundary}\r\nContent-Type: ${mimeType}\r\n\r\n`
  );
  const footer = Buffer.from(`\r\n--${boundary}--`);
  const body = Buffer.concat([header, videoBuffer, footer]);
  return new Blob([body], { type: `multipart/related; boundary=${boundary}` });
}
