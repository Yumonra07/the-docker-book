# Docker images
Docker imajları layer layer çalışır. Base bir imaj üzerine, birden fazla eklemeler yaparak yeni imajlar oluşturabiliriz.

Docker imajlardan başlattığı containerlar için bu layerları load ettikten sonra, üstüne yeni bir layer ekleyerek imaj üzerinde yapılan değişiklikleri "copy on write" tekniği ile container durumunda kayededer.

Bu şekilde imajlardan yeni container başlattığımızda, diğer containerlarda yapılan değişikliklerden etkilenmeyiz. Ve bu containerların durdurulup, çalışması tarzı durumlarda çok işimize yarar.

## Listing docker images
`docker images` komutunu kullanarak docker imajlarımızın hepsini listeleyebiliriz.

`docker pull` kullanarak docker repolarında bulunan imajlardan herhangi birini çekebiliriz.

Aynı zamanda docker çalıştırdığımız build komutlarındada eğer dependent imajlar yoksa otomatik olarak bu imajları indirebilir.

Tüm docker imajlarına tag belirtebiliriz ve bir imaj reposu altında birden fazla tagla imaj bulunabilir.

Örnek vermek gerekirse, docker ubuntu imajları için :12.04, :12.10 gibi imajlı taglar bulabiliriz.

Aynı zamanda bir imaj çalıştırırken çalıştırma komutumuzda bu imajın versiyonunu verebiliriz. Örneğin: `docker run -t -i ubuntu:12.04 bash` gibi.

user/imaj şeklinde isimlendirilen imajlar, docker kullanıcı tarafından yaratılmış olan, herhangi bir onay işleminden geçmemiş olan imajlar için kullanılır. İsteyen herkes docker reposuna kendi imajlarını ekleyebilir.

## Building Docker Images
Docker imaj build etmenin 2 önerilen yöntemi vardır. Bunlardan biri `docker commit` kullanmak, diğeri ise `docker build` komutunu kullanarak bir Docker file ile imaj oluşturmaktır.

### Docker Commit
Docker commit çalıştırdığınız bir imajın containerı üzerinde yaptığınız değişiklikler ile bir imaj oluşturmanızı sağlar. Bu genellikle önerilmeyen bir yöntemdir. Ne kadar oluşturulan imaj lightweight olsada, docker imajını aktarması, paylaşması ve maintain etmesi gibi durumlar zorlaşabilir.

`docker inspect` ile bir imajımız hakkında genel bilgiler edinebiliriz.

`docker commit -m 'Commit message' -a 'Author' $CONTAINER_ID user/image:tag` komutu ile bir docker containerını imaja çevirebiliriz.

### DockerFile
Docker imajlarınızı tekrar edilebilir şekilde oluşturmak e saklamak için Dockerfile adınız verdiğimiz dosyaları oluşturabilirsiniz.

Bu dosyalarda docker için tanımlanmış özel bir DSL ile ard arda çalıştıracağınız komutlar ve belirleyeceğiniz özelliklerle, bir docker imajı oluşturabilirsiniz.

Docker imajı oluşturulma sırasında herhangi bir hata oluşması durumunda, en son kaldığınız yerden devam edebilecek şekilde oluşturulmuş bir imaj ile debugging işlemi yapabilirsiniz.

Docker aynı zamanda uzun komutlara sahip dockerfileları rahatca debug edip, tekrar build edebilmek için `docker build` komutunu çalıştırarak build etmeye çalıştığınız imajlarda bir caching mekanizması kullanmaktadır. Bu caching mekanizması sayesinde, son fail olan komutdan itibaren build işlemine devam edebilirsiniz.

Örnek build komutu `docker build --no-cache -t="yengas/test:latest" .` bu komut şuanki bulunduğumuz klasördeki Dockerfile ile, yeni building mekanizmasındaki cachingi yoksayacak şekilde bir build işlemi başlatır.

## Dockerfile Instructionları

### CMD
Bir docker imajı oluşturulduktan sonra, imajdan çalıştırılan containerların çağırıcağı komutları belirler.

CMD ve RUN komutlarında string olarak verilen çalıştırılacak olan komutların başına /bin/sh eklenerek çalıştırılır. Eğer bash olmadığı bir sistemde çalışıyorsanız veya bash olmadan komut çalıştırmak istiyorsanız, array syntaxını kullanarak CMD ve RUN komutlarını çalıştıraiblirsiniz.

Örnek `CMD echo` ile `CMD ["/bin/sh", "-c", "echo"]` aynı komuta tekabül eder.

### Entrypoint
Docker imajlarınız için standart bir giriş noktası oluşturan komutdur. Bu komut sayesinde imajınızı belli bir program gibi çalışmaya zorlayabilirsiniz.

Örnek vermek gerekirse ENTRYPOINT ["nginx"] yapılan bir docker imajı docker run ile çalıştırıldığı zaman, verilen parametreler bu programın üzerinde çalışır.

ENTRYPOINT ve CMD komutlarını birleştirip, docker runda parametre verilmeyince default olarak çalışan bir komut yazabiliriz.

Örneğin official docker jenkins imajı kendi yarattıkları bir bash scripti entrypoint olarka verip, eğer user -- ile başlayan bir docker run komutu çalıştırırsa daha sonra verilen argümanları java vm'sine parametre olarak kabul ediyor, eğer başında -- olmadan bir komut verirse, direk komutun kendisini çalıştırıyor.

### WORKDIR
Bu komutdan sonra çalıştıracağınız entrypoint, run, cmd komutları için çalışma directorysini set eder.

### ENV
Environment variable set etmenizi sağlar. Bu environment variablelar imajlar üzerinden çalıştıran containerlar içinde geçerli olur.

### USER
Komutları çalıştıracak USER'ı belirler. Ve container oluşturulduğunda başlayacak user'ı set eder.

### VOLUME
Docker containerlarının paylaşabileceği, hostda persistent olarak kullanılabilecek fs mountpointleri oluşturmanıza yarar.

CP komutunu kullanarak docker containerınızdan hosta, veya tam tersi şekilde dosya aktarmanıza yardımcı olur.

### ADD
Build environmentimizden, imajımıza dosya kopyalamamızı sağlar.

Targetın sonu / ile bitiyorsa directory, başka bir karakter ile bitiyorsa normal dosya olduğunu varsayar.

Tar dosyalarını otomatik olarak unzipleyebilir.

### COPY
ADD komutu gibi çalışır fakat sadece build komutunun çalıştığı klasördeki dosyaları kopyalamanızı sağlar. Tüm pathler build pathine relative olmalıdır.

### LABEL
Docker imajlara Key/Value formunda metadata eklemenizi sağlar. Docker inspectde görünmesini sağlar.

### STOPSIGNAL
Docker containera durması için hangi sistem sinyalinin gönderileceğini söyler.

### ARG
Docker imaj build processi için docker build'i çalıştırırken verebileceğiniz komutları ve defaultlarını belirler. Fakat bu arglar imajın build historysinde gözükür, o yüzden gizli bilgileri koymak güvenli değildir.

### ONBUILD
Build esnasında FROM'dan hemen sonra çalışacak komutlar belirtebilmeniz için kullanılır.
`ONBUILD ADD . /var/www/` FROM'dan hemen sonra Dockerfile'ın bulunduğu bütün klasörü, imajın /var/www klasörüne taşır.

## Docker running containers
Docker imajlarından container çalıştırırken dikkat etmeniz gereken bazı şeyler vardır. Bunlardan biri PORT expose mekanizmasıdır. PORT expose makinası, çalıştırdığınız docker imajlarının hangi portunun host makinede hangi porta denk geldiğini belirlemenize yardımcı olur.

Eğer bir port belirlemezsiniz, expose edilen portlar rastgele high port aralığında bir porta assign edilecek ve containerlarınız bu şekilde çalışacaktır.

Isterseniz `docker run` komutunu kullanırken `-p 8080:80` gibi tek tek spesifik olarak imajınızın expose ettiği portları host makinedeki portlara assign edebilirsiniz.

Aynı zamanda `-P` komutunu kullanarakta imajda expose edilen tüm portları hostdaki karşılığına assign edebilirsiniz.
