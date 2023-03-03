## Wakanda-1-YolHaritasi
Çözdüğüm ctf'leri unutmamak amacıyla buraya adımlar şeklinde not alıyorum. Bu ctf'i [*vulnhubdan*](https://www.vulnhub.com/entry/wakanda-1,251/) bilgisayarınıza indirip virtualbox'ınıza kurabilirsiniz. Bu ctf'de elde ettiğimiz önemli bilgileri not.txt dosyasına kaydedeceğiz. Virtualbox'a kurduktan sonra makineyi başlatarak ctf'i çözmeye başlayabiliriz.
### *Adım 1:*
İlk olarak tüm ctf'lerde olduğu gibi zafiyetli makinenin açık portlarına bakarak işe başlamayı düşündüm. Lakin nmap taraması yapmak için bir ip adresine ihtiyacımız var. Zafiyetli makine ile aynı ağda olduğumuz için `netdiscover -r 10.0.2.0/24` çalıştırarak ağımdaki ip'leri aşağıdaki gibi elde edebiliriz.

![](https://github.com/hamza37yavuz/Wakanda-1-YolHaritasi/blob/main/netdiscover.png)

Eğer bağlantı arayüzünüz eth0 değilde wlan0 ise netdiscover komutunun aralığını (range) ona göre ayarlamanız gerekir.
Zafiyetli makinenin ip'sini yukarıdaki gibi elde ettikten sonra `nmap -v -sS -A -T4 <Hedef IP>` komutunu çalıştrarak açık portları görüntüleyebiliriz.

![](https://github.com/hamza37yavuz/Wakanda-1-YolHaritasi/blob/main/nmap.png)

Açık portlara baktığımızda ilk dikkatimizi çeken şey 80 portu olyor. Bir sonraki adımda bu websiteyi mutlaka ziyaret etmemiz gerekir. Burada 3333 portunda ssh açık olduğunu görebiliyoruz. Eğer 80 portunda bir şey çıkmazsa makineye ssh kullanarak varsayılan şifre ve kullanıcılar bağlanmaya çalışabiliriz. 111 portunda da başka bir hizmetin açık olduğunu görüyoruz ama bunu araştırmadan önce 2. adıma geçelim ve web zafiyeti olup olmadığına bakalım.
### *Adım 2:*
Siteye girdiğimizde bizi basit bir ekran karşılıyor. Bu ekranda dikkatimizi çeken ilk şey en altta bulunan mamadou kullanıcı ismi oluyor. Bu isme muhtemelen ssh bağlantısı sırasında ihtiyacımız olacaktır. O yüzden not dosyamıza bunu not ediyoruz.

![](https://github.com/hamza37yavuz/Wakanda-1-YolHaritasi/blob/main/username.png)

Bu site için gobuster komutunu ve dirb'ü ayrı ayrı çalıştırabiliriz nitekim ben çalıştırdım ve kayda değer bir sonuç alamadım. Belki sayfa kaynağında bir şey buluruz 
ümidiyle bakıyoruz. HTML dökümanını incelediğimde bir yorum satırı dikkatimi çekti burada bir link yönlendirmesi bulunuyor:

`<!-- <a class="nav-link active" href="?lang=fr">Fr/a> -->`

Burada ?lang=fr kısmını site linkinin sonuna eklediğimizde sitenin fransızca olduğunu görüyoruz. Site adres kısmında daha önce yazılmış bir php dosyası kulanıldığı çıkarımını yapabiliriz. Bu çıkarıma göre linkin ?lang= sonraki kısmından sistem dosyalarına ulaşma denemeleri yapıyoruz fakat sonuç alamıyoruz. Bu denemeleri nasıl yapacağımızı ve denemeler sonuç vermezse neler yapabileceğimizigörmek için * [bu linkten](https://book.hacktricks.xyz/pentesting-web/file-inclusion) *yararlanabiliriz.
Bu sitede yaptığımız okuma sonucu farklı bir index.php dosyasına erişmek ve php filter'ı atlatmak için ?lang= sonrasına sitede yer alan base64 ile ilgili bir link ekliyoruz:

`php://filter/convert.base64-encode/resource=index` 

Sayfa kaynağına giderek base64 ile şifrelenmiş metni alıyoruz ve bir [online decoder](https://www.base64decode.org/) yardımıyla metnideşifre ediyoruz. Ulaştığımız metinde bir şifre kısmı görüyoruz. Elde ettiğimiz şifreyi not dosyamıza ekliyoruz. Artık bir kullanıcı ismine ve şifreye sahibiz. Bununla beraber ssh portu da açık olduğunu biliyoruz. Kullanıcı adımız mamadou şifremiz 'Niamey4Ever227!!!'. Bağlantı başarılı!!
### * Adım 3: *
ssh ile başarıyla bağlandık ama karşımıza çıkan shell içerisinde linux komutlarını çalıştırdığımızda hata aldığımızı görüyoruz. Hatayı dikkatle incelediğimizde ve farklı denemeler yaptığımızda bunun python hataları olduğunu anlayabiliyoruz.

![](https://github.com/hamza37yavuz/Wakanda-1-YolHaritasi/blob/main/ssh.png)

Burada python çalıştırabildiğimize göre python ile kabuk oluşturabiliriz. Kabuk için 

`import pty; pty.spawn("/bin/bash")`

komutunu çalıştırıyoruz. Artık başta çalıştırmayı denediğimiz tüm komutları çalıştırabiliriz. flag1.txt'yi aşağıdaki gibi elde ediyoruz.

![](https://github.com/hamza37yavuz/Wakanda-1-YolHaritasi/blob/main/flag.png)
