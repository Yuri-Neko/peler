Error? Fix sendiri jangan manja ya bang kamu kan udah gede

import FormData from "form-data"
import Jimp from "jimp"

async function processing(urlPath, method) {
	return new Promise(async (resolve, reject) => {
		let Methods = ["enhance", "recolor", "dehaze"];
		Methods.includes(method) ? (method = method) : (method = Methods[0]);
		let buffer,
			Form = new FormData(),
			scheme = "https" + "://" + "inferenceengine" + ".vyro" + ".ai/" + method;
		Form.append("model_version", 1, {
			"Content-Transfer-Encoding": "binary",
			contentType: "multipart/form-data; charset=uttf-8",
		});
		Form.append("image", Buffer.from(urlPath), {
			filename: "enhance_image_body.jpg",
			contentType: "image/jpeg",
		});
		Form.submit(
			{
				url: scheme,
				host: "inferenceengine" + ".vyro" + ".ai",
				path: "/" + method,
				protocol: "https:",
				headers: {
					"User-Agent": "okhttp/4.9.3",
					Connection: "Keep-Alive",
					"Accept-Encoding": "gzip",
				},
			},
			function (err, res) {
				if (err) reject();
				let data = [];
				res
					.on("data", function (chunk, resp) {
						data.push(chunk);
					})
					.on("end", () => {
						resolve(Buffer.concat(data));
					});
				res.on("error", (e) => {
					reject();
				});
			}
		);
	});
}
let handler = async (m, { conn, usedPrefix, command }) => {
	switch (command) {
		case "unblur":
		case "hd":
		case "hdr":
		case "remini":
			{
				conn.enhancer = conn.enhancer ? conn.enhancer : {};
				if (m.sender in conn.enhancer)
					throw `Masih ada proses yang belum selesai, mohon tunggu beberapa saat...`
				let q = m.quoted ? m.quoted : m;
				let mime = (q.msg || q).mimetype || q.mediaType || "";
				if (!mime)
					throw `Fotonya?`
				if (!/image\/(jpe?g|png)/.test(mime))
					throw `Mime ${mime} tidak support`
				else conn.enhancer[m.sender] = true
				conn.sendMessage(m.chat, { react: { text: `🕑`, key: m.key }})
				let img = await q.download?.();
				let error;
				try {
					const This = await processing(img, "enhance")
					conn.sendFile(m.chat, This, "", null, m)
				} catch (er) {
					error = true
				} finally {
					if (error) {
						m.reply("Proses gagal :(")
					}
					delete conn.enhancer[m.sender]
				}
			}
			break
		case "colorize":
			{
				conn.recolor = conn.recolor ? conn.recolor : {};
				if (m.sender in conn.recolor)
					throw `Masih ada proses yang belum selesai, mohon tunggu beberapa saat...`
				let q = m.quoted ? m.quoted : m;
				let mime = (q.msg || q).mimetype || q.mediaType || "";
				if (!mime)
					throw `Fotonya?`
				if (!/image\/(jpe?g|png)/.test(mime))
					throw `Mime ${mime} tidak support`
				else conn.recolor[m.sender] = true
				conn.sendMessage(m.chat, { react: { text: `🕑`, key: m.key }})
				let img = await q.download?.();
				let error;
				try {
					const This = await processing(img, "recolor")
					conn.sendFile(m.chat, This, "", null, m)
				} catch (er) {
					error = true
				} finally {
					if (error) {
						m.reply("Proses gagal :(")
					}
					delete conn.recolor[m.chat]
				}
			}
			break
		case "dehaze":
			{
				conn.hdr = conn.hdr ? conn.hdr : {};
				if (m.sender in conn.hdr)
					throw `Masih ada proses yang belum selesai, mohon tunggu beberapa saat...`
				let q = m.quoted ? m.quoted : m;
				let mime = (q.msg || q).mimetype || q.mediaType || "";
				if (!mime)
					throw `Fotonya?`
				if (!/image\/(jpe?g|png)/.test(mime))
					throw `Mime ${mime} tidak support`
				else conn.hdr[m.sender] = true
				conn.sendMessage(m.chat, { react: { text: `🕑`, key: m.key }})
				let img = await q.download?.();
				let error;
				try {
					const This = await processing(img, "dehaze")
					conn.sendFile(m.chat, This, "", null, m)
				} catch (er) {
					error = true
				} finally {
					if (error) {
						m.reply("Proses gagal :(")
					}
					delete conn.hdr[m.sender]
				}
			}
			break
	}
}
handler.command = handler.help = ["unblur", "hd", "remini", "colorize", "dehaze"]
handler.tags = ["tools"]
export default handler
