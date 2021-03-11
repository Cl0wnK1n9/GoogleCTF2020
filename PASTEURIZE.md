# GoogleCTF2020 - PASTEURIZE

Đang ngồi lướt lướt youtube xem mấy video của [LiveOverflow](https://www.youtube.com/channel/UClcE-kVhqyiHCcjYwcpfj9w), ông này chủ yếu làm về pwn cực kỳ hay cho những ai theo mảng này (RE và pwn nhé :V ) dù mình không phải pwner nhưng vẫn hay xem vì nó hay, thì thấy có 1 video về cái googleCTF2020 và mảng web ??? Mặc dù đã bị spoil cực mạnh nhưng mình vẫn sẽ làm coi như là đọc writeup rồi làm lại xem mình có hiểu không và đây là writeUp của mình về bài PASTEURIZE còn bài mình xem là bài Tech Support :=) 

# Khám phá 
 Web có 2 chức năng 1 là tạo post và cái thứ 2 là gửi cho "Mi Ke" - một idol giới trẻ. Nhìn qua dạng đề kết hợp với kinh nghiệm vài bữa chơi CTF của mình thì chắc chắn bài này là về client side hay nói rõ hơn là XSS còn XSS loại nào thì chưa biết. 
 
 # Khám
 Một bài về Client side thì về cơ bản chỉ cần bạn bật Ctr+U lên là có thể hình dung được các bước tấn công hoặc chí ít cũng là thu thập được 1 vài thông tin hữu ích cũng như luồng xử lý của chương trình. 

![img1](https://github.com/Cl0wnK1n9/GoogleCTF2020/blob/main/pasteurize/img/2.PNG)

Như bạn thấy nội dung của post sẽ được đưa vào trong script -> Phải break được `"` để thực thi javascript và lấy cookie. Điều thứ 2 chính là cái comment có vẻ có vấn đề gì đó trong source code so let's check it. 

```app.get('/:id([a-f0-9\-]{36})', recaptcha.middleware.render, utils.cache_mw, async (req, res) => {
  const note_id = req.params.id;
  const note = await DB.get_note(note_id);

  if (note == null) {
    return res.status(404).send("Paste not found or access has been denied.");
  }

  const unsafe_content = note.content;
  const safe_content = escape_string(unsafe_content);

  res.render('note_public', {
    content: safe_content,
    id: note_id,
    captcha: res.recaptcha
  });
});
```
Đây là đoạn code render ra cái post mà mình đã up lên, trong đó phần nội dung của post đã được đưa qua hàm `escape_string` để chống XSS. 

```
const escape_string = unsafe => JSON.stringify(unsafe).slice(1, -1).replace(/</g, '\\x3C').replace(/>/g, '\\x3E'); 
```
Như có thể thấy mọi kí tự `<` và `>` đều bị đưa thành `\x3C` và `\x3E` -> Bạn không thể đóng thẻ `script` hiện tại để mở ra thẻ `script` mới được và nếu muốn inject mấy cái payload kiểu \<img src=xxx onerror=alert(1)> thì quên đi vì như bạn thấy DOMPurify.sanitize sẽ loại bỏ tất cả mọi thứ liên quan đến javascript trong cái input đó => HTML injection: YES, XSS: not now.
Tiếp theo, mục đích bây giờ là phải làm sao break được cái string ở đoạn gắn biến `note`=>`const note = "ahihi";`. Đương nhiên cách duy nhất là trong input phải có thế 1 ký tự `"` nhưng không `JSON.stringify` sẽ đưa mọi ký tự `"` trong input thành `\"` nghĩa là nó vẫn được hiểu là 1 ký tự thôi.

# Phá
Đây là quá trình mình tìm cách xử lý cái dấu `"` nhìn nó có vẻ hơi ngắn nhưng mất tận chục phút đọc về `JSON.stringify` và thêm tiếng nữa thử đi thử lại đấy.

![img1](https://github.com/Cl0wnK1n9/GoogleCTF2020/blob/main/pasteurize/img/4.PNG)

Như bạn thấy nếu chỉ là 1 chuỗi thong thường thì `"` chắc chắn sẽ bị đổi thành `\"` (`'` vẫn sẽ được giữ nguyên) còn nếu là 1 mảng ký tự thì ký tự `"` ở vị trí mở chuỗi và đóng chuỗi sẽ được giữ nguyên. Đến đây bài này coi như đã xong việc còn lại chỉ là craft payload sao cho nó chạy nữa thôi 🤟🤟🤟🤟🤟🤟

![img1](https://github.com/Cl0wnK1n9/GoogleCTF2020/blob/main/pasteurize/img/3.PNG)
