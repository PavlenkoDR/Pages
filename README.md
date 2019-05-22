# Дамп картинок конкретного человека через палки и костыли

## 1. Переходим на [https://vk.com/dev/messages.search](https://vk.com/dev/messages.search)
Нам понадобятся:
- Поле q - Тут вбиваем Имя и Фамилию юзверя
- peer_id если хотим получить картинки изконкретного чата. Нужно вбить либо id пользователя, либо 2000000000 + id беседы. Например если номер беседы 72, то получится 200000000072

- date - дата в формате DDMMYYYY — если параметр задан, в ответе будут только сообщения, отправленные до указанной даты. Например 23052019
- count делаем 100

Остальное не трогаем

## 2. Жмем F12 на клавиатуре. 
Открываем вкладку Sources(для Chrome). 
Переходим в top->vk.com->js->al->dev.js.


### Способ 1. Не подходит для хрома

Находим функцию methodRun.
И после кода
```js
...
var res = parseJSON(code);
if (res.error && res.error.error_code == 14) {
  cur.appCaptcha = showCaptchaBox(res.error.captcha_sid, 0, false, {
    onSubmit: function(sid, value) {
      Dev.methodRun(hash, btn, {captcha_sid: sid, captcha_key: value});
      cur.appCaptcha.hide();
    },
    imgSrc: res.error.captcha_img
  });
}
...
```
Вставляем

```js
if (res.response.items != null)
{
	for (var itemIdx in res.response.items)
	{
		var item = res.response.items[itemIdx];
		for (var attachmentIdx in item.attachments)
		{
			var attachment = item.attachments[attachmentIdx];
			if (attachment.type == 'photo' && attachment.photo.sizes != null)
			{
				var photoUrl = attachment.photo.sizes[attachment.photo.sizes.length - 1].url;
				var fileName = photoUrl.substring(photoUrl.lastIndexOf('/') + 1);
				var lastJpgInd = fileName.lastIndexOf('.jpg') + 4;
				fileName = fileName.substring(lastJpgInd, fileName.Length - lastJpgInd);
				console.log(fileName);
				var link = document.createElement("a");
				link.download = fileName;
				link.href = photoUrl;
				link.target = '_blank';
				document.body.appendChild(link);
				link.click();
				document.body.removeChild(link);
				delete link;
			}
		}
	}
}
```

### Способ 2

Находим функцию methodRun.
И после кода
```js
var onResponse = function(code) {
  if (code) {
    code = code.replace(/^<pre>(.*)<\/pre>$/, '$1');
  }
  ...
```
Вставляем
```js
var element = document.createElement('a');
element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(code));
element.setAttribute('download', "filename.txt");

element.style.display = 'none';
document.body.appendChild(element);

element.click();

document.body.removeChild(element);
```
Теперь при каждом клике на кнопку "Выполнить" у нас будет сохраняться JSON

## 3. В консоль браузера вставляем 
```js
var myFunc = function(i)
{
	setTimeout(()=>{document.getElementById('dev_const_offset').value=i*100;document.getElementById('dev_req_run_btn').click();}, i*3000);
}

for(var i = 0; i < 100; i++)
{
    myFunc(i);
}
```

Файлы скачиваются

## 4. Продолжение для Способа 2
Создаем консольный C# проект в Visual Studio 2017
Подтягиваем через NuGet либы: Newtonsoft.Json, Microsoft.CSharp

В файл вставляем код:
```cs
using Newtonsoft.Json;
using System;
using System.IO;
using System.Net;

namespace ConsoleApp1
{
    public class VKJson
    {
        public string access_token { get; set; }
        public string expires_in { get; set; }
        public string user_id { get; set; }
    }

    class Program
    {
        static void Main(string[] args)
        {
            string dir = Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData);
            string jpegsDir = Path.Combine(dir, "jpegs\\");
            string filesDir = Path.Combine(dir, "files\\");
            Directory.CreateDirectory(jpegsDir);
            Directory.CreateDirectory(filesDir);
            foreach (var file in Directory.GetFiles(filesDir))
            {
                var text = File.OpenText(file).ReadToEnd();
                Console.WriteLine();
                Console.WriteLine();
                Console.WriteLine(file);
                Console.WriteLine();
                dynamic json = JsonConvert.DeserializeObject(text);
                var items = json.response.items;
                var fwd_messages = json.response.fwd_messages;
                while (items != null || fwd_messages != null)
                {
                    if (items != null)
                    {
                        foreach (var item in items)
                        {
                            if (item.attachments != null)
                            {
                                foreach (var attachment in item.attachments)
                                {
                                    if (attachment.type == "photo" && attachment.photo.sizes != null)
                                    {
                                        using (WebClient webClient = new WebClient())
                                        {
                                            string photoUrl = attachment.photo.sizes[attachment.photo.sizes.Count - 1].url;
                                            string fileName = photoUrl.Remove(0, photoUrl.LastIndexOf("/") + 1);
                                            int lastJpgInd = fileName.LastIndexOf(".jpg") + 4;
                                            fileName = fileName.Remove(lastJpgInd, fileName.Length - lastJpgInd);
                                            fileName = Path.Combine(jpegsDir, fileName);
                                            Console.WriteLine(fileName);
                                            try
                                            {
                                                webClient.DownloadFile(new Uri(photoUrl), fileName);
                                            }
                                            catch
                                            {
                                                Console.WriteLine("Error");
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                    items = null;
                    if (fwd_messages != null)
                    {
                        Console.WriteLine(">>>>> from forwarded");
                        items = fwd_messages.items;
                        fwd_messages = fwd_messages.fwd_messages;
                    }
                }
            }
        }
    }
}

```
После первого исполения в AppData/Local создадутся папки files и jpegs. В files необходимо закинуть скачанные файлы из второго способа и далее запустить программу.

Картинки будут в папке jpegs
