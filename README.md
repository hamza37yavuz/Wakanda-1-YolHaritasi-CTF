## Wakanda-1-YolHaritasi
Çözdüğüm ctf'leri unutmamak amacıyla buraya adımlar şeklinde not alıyorum. Bu ctf'i [*vulnhubdan*](https://www.vulnhub.com/entry/wakanda-1,251/) bilgisayarınıza indirip virtualbox'ınıza kurabilirsiniz. Virtualbox'a kurduktan sonra makineyi başlatarak ctf'i çözmeye başlayabiliriz.
### *Adım 1:*
İlk olarak tüm ctf'lerde olduğu gibi zafiyetli makinenin açık portlarına bakarak işe başlamayı düşündüm. Lakin nmap taraması yapmak için bir ip adresine ihtiyacımız var. Zafiyetli makine ile aynı ağda olduğumuz için `netdiscover -r 10.0.2.0/24` çalıştırarak ağımdaki ip'leri aşağıdaki gibi elde ettim.

![](https://github.com/hamza37yavuz/Wakanda-1-YolHaritasi/blob/main/netdiscover.png)

Eğer bağlantı arayüzünüz eth0 değilde wlan0 ise netdiscover komutunun aralığını (range) ona göre ayarlamanız gerekir.
Zafiyetli makinenin ip'sini yukarıdaki gibi elde ettikten sonra `nmap -v -sS -A -T4 <Hedef IP>` komutunu çalıştrarak açık portları görüntüleyebiliriz.

![](https://github.com/hamza37yavuz/Wakanda-1-YolHaritasi/blob/main/nmap.png)

Açık portlara baktığımızda ilk dikkatimizi çeken şey 80 portu olyor. bir sonraki adımda bu websiteyi mutlaka ziyaret etmemiz gerekir. Burada 3333 portunda ssh açık olduğunu görebiliyoruz. Eğer 80 portunda bir şey çıkmazsa makineye ssh kullanarak varsayılan şifre ve kullanıcılar bağlanmaya çalışabiliriz. 111 portunda da başka bir hizmetin açık olduğunu görüyoruz ama bunu araştırmadan önce 2. adıma geçelim ve web zafiyeti olup olmadığına bakalım.
### *Adım 2:*
Siteye girdiğimizde bizi basit bir ekran karşılıyor. Bu ekranda dikkatimizi çeken ilk şey en altta bulunan mamadou kullanıcı ismi oluyor.

![](https://github.com/hamza37yavuz/Wakanda-1-YolHaritasi/blob/main/username.png)

Bu site için gobuster komutunu ve dirb'ü ayrı ayrı çalıştırabiliriz nitekim ben çalıştırdım ve kayda değer bir sonuç alamadım. En iyisi sayfa kaynağına bakmamız olacaktır. HTML dökümanını incelediğimde bir yorum satırı dikkatimi çekti burada bir link yönlendirmesi bulunuyor:

