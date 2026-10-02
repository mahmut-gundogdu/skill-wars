# Skill Savaşları: mattpocock/skills ve obra/superpowers

Kodlama ajanları için iki popüler skill kütüphanesinin, Anthropic ve OpenAI'nin 2026 tarihli skill ve prompt yazma rehberlerine göre, geliştirici gözüyle karşılaştırması. İngilizce sürüm: [README.md](README.md).

| | [mattpocock/skills](https://github.com/mattpocock/skills) | [obra/superpowers](https://github.com/obra/superpowers) |
|---|---|---|
| İncelenen commit | `d81f3a1` (2026-09-29, v1.2.3) | `8ca22db` (2026-09-25, v6.4.2) |
| Ne olduğu | Çoğunu elle çağırdığınız 37 küçük skill'lik bir araç kutusu (27'si dağıtımda) | 15 skill'den oluşan, zorunlu, uçtan uca bir geliştirme metodolojisi |
| Skill'ler nasıl tetiklenir | Yalnızca açıklama metniyle. Dağıtılan 27 skill'in 16'sı ancak siz yazınca çalışır | Her oturumda bir hook, modele skill kullanmasını emreden bir bootstrap metni enjekte eder |
| Her zaman yüklü bağlam maliyeti | 2.150 karakter açıklama | ~900 token bootstrap artı 2.689 karakter açıklama; her `/clear` ve compaction sonrası yeniden enjekte edilir |
| SKILL.md boyutu | medyan 534 kelime, en fazla 160 satır, 500 satırı aşan yok | medyan 1.246 kelime, 500 satırı aşan iki skill (564 ve 677) |
| Referans dosyaları | 23 dosya, 11,4 bin kelime | 45 dosya, 27,8 bin kelime |
| Bağırma (BÜYÜK HARF vurgu kelimeleri) | 8 | 153 |
| Rasyonalizasyon ("bahane → gerçek") satırları | 0 | 98, 15 skill'in 12'sinde |
| Bir özellik akışında insan onayı kapıları | 2 ila 4, hepsi sizin yazdığınız | en az 7 (toplam 19 ayrı kapı var) |
| Depo içi davranış eval'leri | yok | kısmi: depo içi LLM testleri artı harici bir eval deposu (incelenmedi) |
| Harness desteği | Claude Code plugin'i, diğerleri `npx skills` ile | Yerel plugin'lerle 16 harness |
| Kurulum | Her repoda bir kez `/setup-matt-pocock-skills`, bir issue tracker yapılandırması | Kur ve kullan |

## Tek paragrafta karar

İki kütüphane de aynı mühendislik kanonunu kodlar (yazmadan önce görüşme, spec, kırmızı-yeşil TDD, kök neden ayıklama, alt-ajanla inceleme, worktree'ler) ama modele dair zıt kuramlarla. **superpowers** modeli disipline edilmesi gereken bir şey olarak görür: oturum başı bootstrap, dört BÜYÜK HARFLİ "Demir Yasa", modelin kendi bahanelerine karşı önceden yazılmış 98 satır çürütme ve her tasarım adımında onay kapıları. Bu, Anthropic ve OpenAI rehberlerinin artık açıkça uyardığı tarzdır ve depo bunu kendisi de söyler: katkı rehberi felsefesinin "Anthropic'in yayımlanmış rehberinden farklı olduğunu" belirtir. **mattpocock/skills** modeli yetkin kabul eder ve indeksi insanda tutar: skill'ler kısadır, vurgu büyük harf yerine kalın kelimelerle taşınır ve çoğu skill siz yazana kadar hiçbir bedel ödetmez. Zayıf tarafı orantılılıktır: `tdd`, `diagnosing-bugs` ve görüşme skill'leri küçük işlerde küçülmeyen sabit bir tören dayatır ve depoda hiç eval yoktur. İki kütüphane de "her modelle çalıştığını" kanıtlamaz. Fikri belli bir otopilot istiyor ve bedelini kabul ediyorsanız superpowers'ı seçin. Kendi kontrolünüzde, birleştirilebilir araçlar istiyorsanız mattpocock/skills'i seçin ve kendiliğinden tetiklenen iki disiplin skill'ini kullanıcı tetiklemeli yapın.

## Yöntem

- Her depo klonlandı ve ortak bir brief ile değerlendirme ölçütüne göre ayrı bir Opus 5.5 ajanı tarafından incelendi. Her sayı bir betikle üretildi, her iddia bir `dosya:satır` taşıyor. Orkestratör iki rapordaki 28 alıntıyı dosyalara karşı doğruladı (hepsi eşleşti), ardından her ajana akışlar, kapılar, kanıtlar ve karşı tarafın en güçlü savunması üzerine yedi doğrulama sorusu sordu; her cevap dosyalarla tutarlıydı. Tam raporlar [reports/](reports/) altında.
- Ölçütler: Anthropic'in [Claude Code en iyi uygulamaları](https://code.claude.com/docs/en/best-practices), Anthropic'in [Skill yazma en iyi uygulamaları](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) ve OpenAI'nin [Rethinking skills and prompts for GPT-6 Astra](https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra) yazısı. OpenAI yazısı analiz ortamının ağ politikası tarafından engellendi; önerileri arama sonuçları ve basın haberlerinden yeniden kuruldu ([reports/openai-gpt6-astra-reconstructed.md](reports/openai-gpt6-astra-reconstructed.md)) ve ikincil kaynak olarak alıntılandı.
- Rehberler on ölçüte indirgendi: kısa gövde, belirgin açıklama, kırılganlığa uygun serbestlik derecesi, kademeli açma (progressive disclosure), "bitti" tanımı, ölçülü vurgu, orantılı tören, eval'ler, tutarlılık, taşınabilirlik. Puanlar analistlerin 1 ile 5 arası yargısıdır ve iki depo arasında ancak kabaca karşılaştırılabilir.

## Her kütüphane nedir

**mattpocock/skills.** Kovalara (engineering, productivity, misc, in-progress) ayrılmış bir koleksiyon. Tek tasarım ekseni bir skill'i kimin çağırabildiğidir: kullanıcı tetiklemeli skill'ler orkestrasyon yapar (`/grill-me`, `/to-spec`, `/to-tickets`, `/implement`, `/triage`, `/wayfinder`) ve modelin erişimi dışındadır; model tetiklemeli skill'ler yeniden kullanılabilir disiplini taşır (`grilling`, `tdd`, `code-review`, `diagnosing-bugs`, `codebase-design`, `pr`). Hiçbir şey birbirini zincirlemez; sonraki adımı siz yazarsınız. `ask-matt` yönlendirici skill'i bir yol önerir ve durur. Bir kurulum skill'i, mühendislik skill'lerinin okuduğu repo başına issue tracker yapılandırmasını ve sözlük düzenini yazar. Bakımcının kendi `writing-for-agents` skill'i Anthropic ve OpenAI önerilerinin bağımsız bir türetimi gibi okunur: etkisiz cümleleri avla, dallara göre aç, her adımı bir tamamlanma ölçütüyle bitir, olumluyu söyle.

**obra/superpowers.** Bir metodoloji. SessionStart hook'u, `<EXTREMELY_IMPORTANT>` ile sarılmış `using-superpowers` skill'inin tamamını enjekte eder; bu metin modele "bir skill'in uygulanma ihtimali %1 bile olsa" onu "KESİNLİKLE ÇAĞIRMASI GEREKTİĞİNİ", hem de açıklayıcı sorular dahil herhangi bir yanıttan önce yapmasını söyler. Akış: brainstorming (bölüm bölüm onaylanan tasarım, yazılı spec), writing-plans, bir worktree, sonra ya alt-ajan güdümlü geliştirme (her görev için bir uygulayıcı ve bir gözden geçirici alt-ajan, en fazla beş düzeltme turu, sonda tüm dalın incelenmesi) ya da satır içi yürütme; TDD, verification-before-completion ve sonda bir birleştirme menüsü. Hiçbir şey hook ile zorlanmaz; tüm zorlama düz yazıyla yapılır. Proje, yazarlara bir skill'i önce taban çizgisiyle test etmeyi, sonra rasyonalizasyon tabloları ve kırmızı bayrak listeleriyle "kurşun geçirmez" yapmayı öğreten bir `writing-skills` skill'i de içerir.

## Benzerlikler

- Aynı kanon: önce görüşme, spec, yeşilden önce kırmızı ile TDD, düzeltmeden önce kök neden, taze bir alt-ajanla inceleme, paralel iş için worktree'ler, YAGNI ve derin modüller.
- İkisi de skill yazma üzerine bir meta-skill dağıtır; Anthropic'in skill dokümanı buna gerek olmadığını söyler.
- İkisi de kendi aşırı tetiklenme sorunlarını belgeler: superpowers'ta negatif tetikleme testi yok; mattpocock'un dokümanları `diagnosing-bugs`'ın sıradan sorun tariflerinde ateşlendiğini kaydeder.
- İkisinde de README ile skill'ler arasında kayma var: superpowers'ın README'si plan okuyucusunu hâlâ v6.4.2'de skill'den çıkarılmış "zevksiz bir junior mühendis" olarak anlatır, hâlâ "2-5 dakikalık görevler" ve iki aşamalı inceleme vaat eder; mattpocock'un yönlendiricisi silinmiş bir devir teslime yönlendirmeye devam eder, `tdd` açıklaması "red-green-refactor" derken gövde refactoring'in döngünün parçası olmadığını söyler.
- İkisi de toplamda kabaca 25 bin kelime SKILL.md metni taşır; fark, bunun ne kadarının tek bir oturuma ulaştığıdır.
- İkisinin de skill'lerinin modeller arası çalıştığına dair kanıtı yok. mattpocock'un dokümanları zayıf modellerin grilling kapısını kırdığını ve GPT-5.6-Sol'un tanıyı aşırı tetiklediğini kaydeder; superpowers'ın kendi notları Codex bootstrap'inin "deneyimi kötüleştirdiği" için kaldırıldığını kaydeder.

## Farklar

| Eksen | mattpocock/skills | obra/superpowers |
|---|---|---|
| Model kuramı | Yetkin; "noktayı daha sert bastırmak ajanın yaratıcılığını az kazanç karşılığında kısıtlar" | Disipline edilmeli; "SEÇME ŞANSIN YOK. KULLANMAK ZORUNDASIN." |
| Zorlama | Tavsiye, insan sıralı, hook yok | Bootstrap hook artı "Zorunlu iş akışları, öneri değil" |
| Vurgu tarzı | Kalın öncü kelimeler; 0 `IMPORTANT`, 0 BÜYÜK HARF `ALWAYS`/`NEVER`/`MUST` | 4 Demir Yasa, 39 kırmızı bayrak maddesi, 98 çürütme satırı, XML tarzı `<HARD-GATE>` etiketleri |
| Küçük iş yolu | Bir yazım hatasında hiçbir skill ateşlenmez; maliyet 2.150 karakter | Çıkış yok; kırmızı bayrak tablosu "skill fazla kaçar" düşüncesine "Kullan" diye cevap verir |
| Yeni özellikte tören | "Sınır boşalana kadar" görüşme (46 soru "olağan"), spec, ticket'lar | Niyet, mesaj başına tek soru, tek tek onaylanan tasarım bölümleri, spec kapısı, plan kapısı, worktree izni, birleştirme menüsü |
| Yürütme | Ticket başına `/implement` ve aralarda `/clear`, ya da worktree'lerde paralel alt-ajanlarla `implement-spec` | Plan onaylandıktan sonra bilerek düşük sürtünme: yalnızca dört durma sınıfı, kararlar kayda geçer |
| Doğrulama | `diagnosing-bugs` ve `wizard`'da güçlü, başka yerde zayıf (E ölçütü ortalaması 3,6) | Baştan sona güçlü: kanıt tabloları, "taze doğrulama kanıtı olmadan tamamlandı iddiası yok" (E ortalaması 3,8) |
| Eval'ler | Yok; "kontrol elle bir çalıştırmadır" | Yeni değişiklikler için gerçek eval disiplini (kontrol grupları, kol başına 5 ila 25 koşu); dondurulmuş içeriğin tek bir destekleyici verisi var |
| Kurulum yükü | Kurulum skill'i, tracker kimlik doğrulaması (`gh`/`glab`), elle oluşturulan etiketler | Kurulum dışında yok |
| Açıklama kuralı | Model tetiklemeli: ne artı ne zaman; kullanıcı tetiklemeli: model bağlamı dışında tutulan tek satırlık insan metni | "YALNIZCA ne zaman kullanılacağını anlatır (NE yaptığını DEĞİL)"; Anthropic'in "ne ve ne zaman" kuralıyla çelişir; kendi 15 açıklamasından 5'i zaten kuralı çiğner |
| Anthropic rehberine duruş | Bağımsız olarak aynı yere varır | Açık, bilinçli ayrışma; "uyum sağlayan" PR'lar reddedilir |

## Dört endişe

### 1. Fazla uzun mu?

**mattpocock: çoğunlukla hayır.** Hiçbir gövde 160 satırı aşmaz; bir planlama oturumu yaklaşık 4 bin, ticket başına bir oturum yaklaşık 3 bin token yükler. Sorunlu noktalar yerel: `wayfinder` tek dosyada 1.954 kelimedir ve temel kuralı ("bir oturumda birden fazla ticket çözme") 105. satırdadır; `ask-matt` her skill'i 1.895 kelimeyle yeniden anlatır; 100 satırı aşan dört referans dosyasında içindekiler yoktur.

**superpowers: evet, iş akışı başına.** Bootstrap tek başına, deponun kendi 150 kelimelik hedefine karşı 520 kelimedir ve her başlangıç, clear ve compaction'da ödenir. Bir özellik akışı denetleyiciye yaklaşık 12.000 kelime skill metni yükler; iki skill 500 satırlık rehber sınırını aşar; 15 skill'in 13'ü deponun kendi 500 kelime hedefini aşar; brainstorming aynı akışı üç kez kodlar (kontrol listesi, grafik, düz yazı); 100 satırı aşan 15 referans dosyasında içindekiler yoktur, Anthropic rehberinin 1.150 satırlık bir kopyası dahil.

### 2. Modeli "aptallaştırıyor" mu?

**mattpocock: düşük.** Hiçbir yerde rasyonalizasyon tablosu, kırmızı bayrak listesi ya da tehdit yok. Kalıntı, teşvik cümleleri ("Agresif ol. Yaratıcı ol. Pes etme." ve ardından standart yeniden üretme tekniklerinin listesi) ve `teach` içindeki tek bir "Parametrik bilgine asla güvenme" cümlesi.

**superpowers: evet, eski katmanda.** `test-driven-development` testten önce yazılmış kod için "Sil. Baştan başla. Ona bakma." der. `using-superpowers` modelin düşüncelerini denetler: "Önce daha fazla bağlama ihtiyacım var" bir rasyonalizasyon olarak listelenir. `verification-before-completion` "Yorgunluk ≠ mazeret" diye açıklar. `writing-skills` skill'i yazarlara ikna araştırmasını (modellere sakıncalı isteklere uymasını sağlama üzerine bir çalışma) uygulamayı öğretir ve modelin bir skill'in yanlış olduğunu savunmasını test başarısızlığı sayar. Yeni katman tam tersidir: v6.4.2 `writing-plans` yetkin bir okuyucu tanımlar, yürütme skill'leri modelden kendi kararlarını verip kaydetmesini ister.

### 3. Eski teknikler mi?

**mattpocock: bağırmada düşük, yasaklarda orta.** 24 bin kelimede 8 büyük harf vurgusu. Ama 110 "don't/never/avoid" ifadesi, yasaklı kelime listeleri ve altı maddelik bir "Yapma" listesi, bakımcının kendi "yasakla yönlendirmek yasaklanan davranışı bağlama sürükler" kuralıyla çelişir.

**superpowers: ağırlıklı olarak evet.** Her zaman yüklü metinde BÜYÜK HARFLİ cümlelerle çift sarılmış `EXTREMELY-IMPORTANT` blokları, "%1 kuralı", dört Demir Yasa, 98 çürütme satırı ve "NEVER fix symptom" ifadesinin bilerek dört kez tekrarlandığını kaydeden bir oluşturma günlüğü. Bakımcılar bunu eval'lerle ayarlanmış olarak savunur. Depo bunu tek bir yerde destekler: TDD'nin çürütme metnini silmek test-önce uyumunu Claude ve Codex'te 8/10'dan 5/10'a düşürmüş. Depoda bootstrap'in büyük harflerini, %1 kuralını ya da "human partner" dilini sınayan hiçbir şey yok; deponun kendi verisi de iki kez güncel modeller için rehberliğin gereksiz olduğunu bulmuş.

### 4. Bürokrasi ve aşırı kısıtlama?

**mattpocock: ana zayıflık.** `tdd`, insanın onaylamadığı bir seam'de test yazmaz, "kullanıcı başında değil" kaçışı yoktur ve kimsenin onay veremeyeceği `implement-spec` arka plan alt-ajanlarının içinde de çalıştırılır; `diagnosing-bugs` altı kapılı faz çalıştırır ve geniş tetikleyicisi ("bir şey bozuk/hata atıyor/başarısız/yavaş") sıradan hata raporlarında ateşlenmesine yol açar; grilling'in soru sınırı yoktur ve README her değişiklikte kullanılmasını söyler; `to-spec` "son derece kapsamlı" kullanıcı hikâyeleri ister. Bakımcı her şikâyeti belgeler (issue #746, #607, #578) ama düzeltmemiştir. Hasarı sınırlayan: dağıtılan 27 skill'in 16'sı yazılmadıkça asla ateşlenmez, bootstrap yoktur ve yönlendirici spec ile ticket'ları çok oturumlu işlere ayırır.

**superpowers: önde yüksek, arkada düşük.** Bir React todo listesi "mimari" olarak sınıflandırılır ve koddan önce ve sonra en az yedi gidiş-dönüş gerektirir. Tek dosyalık bir hata düzeltmesi bile "sınırlı" sayılır ve sınırlı bir işin onayı "mimari bir iş kadar sert bir kapıdır". "İki yol arasında kararsızsan ağır olanı seç. İş ortasında hiçbir şey aşağı inmez." Vazgeçmek, insanın bunu açıkça söylemesini gerektirir. Claude Code rehberinin önerdiği "tek cümlelik diff, planı atla" çıkışı yoktur. Plan onaylandıktan sonra yürütme bilerek kesintisizdir; bu rehberle iyi örtüşür.

## Puan kartı (1 = rehberi yok sayar, 5 = uyar)

| Ölçüt | mattpocock (27 dağıtılan) | superpowers (15) |
|---|---|---|
| A. Kısa gövde | 3,9 | 2,5 |
| B. Açıklama kalitesi | 3,8 | 3,4 |
| C. Serbestlik derecesi | 4,2 | 2,7 |
| D. Kademeli açma | 4,2 | 3,7 |
| E. "Bitti" tanımı | 3,6 | 3,8 |
| F. Vurgu ve iskele | 4,3 | 2,7 |
| G. Orantılılık | 3,7 | 2,4 |
| H. Eval disiplini | 1,6 | 2,7 |
| I. Tutarlılık | 3,9 | 3,3 |
| J. Taşınabilirlik | 3,5 | 3,8 |

En iyi skill'ler: mattpocock `wizard`, `to-questionnaire`, `grilling`, `domain-modeling`; superpowers `diagnosing-superpowers`, `finishing-a-development-branch`, `using-git-worktrees` ve v6.4.2 `writing-plans`. En kötüler: mattpocock `wayfinder`, `diagnosing-bugs`, `teach`; superpowers `using-superpowers`, `brainstorming`, `systematic-debugging`, `writing-skills`.

## Geliştirici olarak ne elde edersiniz

| | mattpocock/skills | obra/superpowers |
|---|---|---|
| Ürünler | GitHub, GitLab ya da yerel dosyalarda spec'ler ve bağımlılık sıralı ticket'lar; triage durum makinesi; mimari HTML raporu; PR gövdeleri; bash kurulum sihirbazları; kaynaklı araştırma notları; sözlük ve ADR'ler | `docs/superpowers/` altında tasarım spec'leri ve planlar; görev başına inceleme defterleri; mockup'lar için tarayıcı tabanlı "görsel yoldaş"; temizlenmiş hata paketiyle oturum adli analizi |
| Orkestrasyon | Worktree'lerde paralel uygulayıcılar (`implement-spec`); paralel alt-ajanlarla iki eksenli inceleme | Model kademelendirmesi ve beş turluk devre kesiciyle görev başına uygulayıcı artı gözden geçirici alt-ajan; sonda tüm dal incelemesi; yedi analistli tanı |
| Deterministik zorlama | Plugin'de hiçbir şey (isteğe bağlı git-guardrail ve pre-commit kurucuları var) | Hiçbir şey; yalnızca düz yazı |
| Size kalan | Skill'leri seçmek ve sıralamak, ticket'ları kapatmak, inceleme bulgularını doğrulamak, TDD ya da tanının değip değmeyeceğine karar vermek | Her tasarım bölümünü, spec'i, planı ve birleştirmeyi onaylamak; gerisi çalışır |
| Bilinen boşluklar | `code-review` alt-ajanları skill'i yeniden çağırabilir (bir rapor 50+ ajana ulaşmış); `implement-spec`'te başarısız uygulayıcı ya da birleştirme çakışması için yol yok | Bayat README ve hatırlama testleri; `requesting-code-review` başka bir skill'in yasakladığı `HEAD~1`'i kullanır; skill'lerin gereksiz yere ateşlenmesini sınayan test yok |

## Öneriler

- **superpowers'ı seçin**, sizi görüşmeye alan, planlayan ve sonra saatlerce gözetimsiz çalışan bir otopilot istiyorsanız ve alt-ajan destekli bir harness kullanıyorsanız. Oturum başına yaklaşık 900 token bootstrap ve uzun bir tasarım evresi bekleyin. Fork'larsanız ilk kesilecek yer bootstrap: harekete geçmeden önce çağır kuralını, iki yönlendirme örneğini, platform işaretçilerini ve "kullanıcı talimatları önceliklidir" satırını tutun (yaklaşık 130 kelime); `EXTREMELY-IMPORTANT` bloğunu ve kırmızı bayrak tablosunu atın.
- **mattpocock/skills'i seçin**, sıfıra yakın bağlam vergisiyle elle birleştireceğiniz küçük araçlar istiyorsanız. Kurulum skill'ini çalıştırın, sonra `tdd` ve `diagnosing-bugs` gerekmeyen işlerde ateşleniyorsa yerelde `disable-model-invocation: true` ayarlayın ve dokümanların önerdiği CLAUDE.md satırlarını ekleyin ("tek seferde bir soru sor").
- **Her ikisi için**: iki bakımcı da ölçmedikleri modeller arası davranış iddiasında bulunuyor. Kullandığınız modellerde test edin. Rehberin kendi döngüsü geçerli: önce skill'siz taban çizgisi, sonra yalnızca davranışı değiştireni ekleyin.
- **Rehberin ikisinden de keseceği şeyler**: yasak listeleri, kapsamlı spec'ler ve yetkin bir modelin zaten varsayılan olarak uyduğu her talimat.

## Dosyalar

- [reports/mattpocock-skills.md](reports/mattpocock-skills.md) ve [reports/obra-superpowers.md](reports/obra-superpowers.md): envanter tabloları, `dosya:satır` ile sorunlu noktalar, kararlar ve her sayıyı üreten betiklerle tam analist raporları (İngilizce).
- [reports/brief.md](reports/brief.md): iki analistin çalıştığı brief ve değerlendirme ölçütü.
- [reports/openai-gpt6-astra-reconstructed.md](reports/openai-gpt6-astra-reconstructed.md): kaynak uyarısıyla yeniden kurulmuş OpenAI önerileri.
