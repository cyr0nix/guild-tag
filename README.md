Discord sunucu etiketleri ile ilgili bir otomatik komut dosyasını Türkçe'ye çevirmekten memnuniyet duyarım. Metin içeriğini (script kısmı hariç) Türkçe'ye çeviriyorum:

# Etiketli Discord Sunucusu Elde Etmek İçin Otomasyon Betiği

6 Mayıs 2025 tarihinden itibaren, yeni oluşturulan Discord sunucularının küçük, rastgele bir şansı (yaklaşık 500'de 1) deneysel etiket özelliğini içermektedir; bu betik, böyle bir sunucuyu bulmak için tekrarlayan sunucu oluşturma ve silme sürecini otomatikleştirir.

> [!NOT]
> Bu özelliğe sahip bir sunucu elde etmek sadece ilk adımdır; etiket işlevini gerçekten *kullanabilmek* için sunucuyu 3 kez boost'lamanız gerekir

Betiği çalıştırmadan önce, aşağıdaki adımları tamamladığınızdan emin olun:

1.  **2FA'yı Devre Dışı Bırakın**: Betik, onay istemeden geçici sunucuları otomatik olarak silebilmek için 2FA'nın devre dışı bırakılmasına ihtiyaç duyar. Betiği kullanmayı bitirdikten sonra 2FA'yı yeniden etkinleştirin.
2.  **Vencord'u Yükleyin**: Bu betik, Vencord istemci modifikasyonu tarafından sunulan fonksiyonlara dayanır. Resmi kaynaktan indirip yükleyin.
3.  **Vencord Eklentisini Etkinleştirin**: Discord Ayarlarını açın, Vencord "Eklentiler" sekmesine gidin, `ConsoleShortcuts` eklentisini bulun ve etkinleştirin. Bu eklenti, geliştirici konsoluna yararlı kütüphane fonksiyonlarını (`findByProps` gibi) sunar, bu da betiğin çalışması için gereklidir.
4.  **Discord İstemcisini Yeniden Başlatın**: Vencord ve eklentinin doğru şekilde yüklendiğinden emin olmak için Discord'u tamamen kapatıp yeniden açın.
5.  **Manuel Olarak Bir Sunucu Oluşturun (Gerekli)**: Betiği çalıştırmadan önce, normal Discord arayüzü üzerinden tek bir sunucu oluşturduğunuzdan emin olun. Bu gerekli bir adımdır.

## Talimatlar

1.  **Geliştirici Konsolunu Açın**: Geliştirici konsolunu açmak için standart tarayıcı/Electron yöntemini kullanın (genellikle Windows/Linux'ta `Ctrl+Shift+I` veya Mac'te `Cmd+Option+I`, bazen F12).
2.  **Yapıştırmaya İzin Verin (İstenirse)**: Konsol, kod yapıştırmadan önce açıkça `allow pasting` yazmanızı isteyebilir.
3.  **Kodu Yapıştırın ve Çalıştırın**: Aşağıdaki JavaScript kod bloğunun tamamını kopyalayın, konsola yapıştırın ve Enter tuşuna basın.

**Betik bir sunucu bulduğunda veya durdurmaya karar verdiğinizde hesabınızda 2FA'yı yeniden etkinleştirmeyi unutmayın.**

---

## Betik

> [!DİKKAT]
> ## **🚨 UYARI: AKTİF YASAKLAMA UYGULAMASI 🚨**
> * **Discord, bu betiği kullanan hesaplara (geçici ve potansiyel olarak kalıcı) yasaklamalar uygulamaktadır. Bu, Hizmet Şartları'nın doğrudan ihlalidir.**  
> * **BU BETİĞİ DEĞER VERDİĞİNİZ HERHANGI BİR HESAPTA KULLANMAYIN. Devam eden uygulama nedeniyle hesabınızın işaretlenme ve yasaklanma riski son derece yüksektir.**  
> * Bu kodu yapıştırıp çalıştırarak, geri dönüşü olmayan yasaklamalar da dahil olmak üzere oluşabilecek tüm hesap işlemleri için bu önemli tehlikeleri kabul ettiğinizi ve tam kişisel sorumluluk üstlendiğinizi kabul edersiniz.  
> * **Kodu YALNIZCA bu ciddi sonuçları tam olarak anlayıp kabul ediyorsanız genişletin**

<details>

<summary> <b>BETİK (genişletmek için tıklayın)</b> </summary>

```js
const BASE_INTERVAL = 120_000;
const DELETE_DELAY = 2_000;
const MAX_RANDOM_ADDITIONAL_DELAY = 3_000;

const SERVER_NAME = "Tag server";
const STOP_ON_FOUND = true; 

console.clear();

function murmurhash3_32_gc(e, _) {
  // no im not gonna use the discord's own hash function

  let $ = (_ = _ || 0),
    c,
    l = new TextEncoder(),
    t = l.encode(e),
    u = t.length,
    i = Math.floor(u / 4),
    m = new DataView(t.buffer, t.byteOffset);
  for (let b = 0; b < i; b++) {
    let n = 4 * b;
    ($ ^= c =
      Math.imul(
        (c =
          ((c = Math.imul((c = m.getUint32(n, !0)), 3432918353)) << 15) |
          (c >>> 17)),
        461845907
      )),
      ($ = Math.imul(($ = ($ << 13) | ($ >>> 19)), 5) + 3864292196),
      ($ >>>= 0);
  }
  c = 0;
  let f = 4 * i;
  switch (3 & u) {
    case 3:
      c ^= t[f + 2] << 16;
    case 2:
      c ^= t[f + 1] << 8;
    case 1:
      (c ^= t[f + 0]),
        ($ ^= c =
          Math.imul(
            (c = ((c = Math.imul(c, 3432918353)) << 15) | (c >>> 17)),
            461845907
          ));
  }
  return (
    ($ ^= u),
    ($ ^= $ >>> 16),
    ($ = Math.imul($, 2246822507)),
    ($ ^= $ >>> 13),
    ($ = Math.imul($, 3266489909)),
    ($ ^= $ >>> 16) >>> 0
  );
}

{
  // Check if the required functions are available
  if (typeof findByProps !== "function") {
    throw new Error(
      "Essential function `findByProps` is missing. Please ensure the 'ConsoleShortcuts' Vencord plugin is installed and enabled."
    );
  }
  if (!findByProps("createGuildFromTemplate")) {
    throw new Error(
      "Could not find the `createGuildFromTemplate` function. Create a server manually once and then try running the script again."
    );
  }
}

const deleteGuild = findByProps(
  "deleteGuild",
  "bulkAddMemberRoles"
).deleteGuild;
const createGuildFromTemplate = findByProps(
  "createGuildFromTemplate"
).createGuildFromTemplate;

class GuildCreator {
  constructor() {
    this.keepRunning = true;
  }

  isInExperimentRange(guild) {
    let hash = murmurhash3_32_gc(`2025-02_skill_trees:${guild.id}`) % 10000;
    return (hash >= 10 && hash < 20) || (hash >= 60 && hash < 100);
  }

  async processGuildCycle() {
    if (!this.keepRunning) {
      console.log("Script instructed to stop. Exiting guild creation cycle.");
      return;
    }

    console.log("Attempting to create a new guild...");
    try {
      const newGuild = await createGuildFromTemplate(
        SERVER_NAME,
        null,
        {
          id: "CREATE",
          label: "Create My Own",
          channels: [],
          system_channel_id: null,
        },
        false,
        false
      );

      if (!newGuild || !newGuild.id) {
        console.error("Failed to create guild.");
        // Schedule next attempt even if this one failed
        if (this.keepRunning) {
          this.scheduleNextCycle();
        }
        return;
      }

      console.log(`Guild created: ${newGuild.name} (ID: ${newGuild.id})`);
      if (this.isInExperimentRange(newGuild)) {
        console.log(
          `🎉 FOUND GUILD WITH TAG: ${newGuild.name} (ID: ${newGuild.id}) 🎉`
        );
        if (STOP_ON_FOUND) {
          console.log("Stopping script as a guild with a tag has been found.");
          this.keepRunning = false;
          return;
        }
        console.log("Guild with tag found, finding more guilds with a tag...");
        this.scheduleNextCycle();
        return;
      } else {
        console.log(
          `Guild (ID: ${newGuild.id}) does not have the tag experiment. Scheduling deletion...`
        );
        setTimeout(async () => {
          console.log(`Deleting guild: ${newGuild.name} (ID: ${newGuild.id})`);
          try {
            await deleteGuild(newGuild.id);
            console.log(`Guild (ID: ${newGuild.id}) deleted.`);
          } catch (err) {
            console.error(`Error deleting guild (ID: ${newGuild.id}):`, err);
          }
        }, DELETE_DELAY + Math.random() * MAX_RANDOM_ADDITIONAL_DELAY);
        
        // Schedule next cycle after scheduling delete
        this.scheduleNextCycle();
      }
    } catch (error) {
      console.error("Error in processGuildCycle:", error);
      // Continue to next cycle even if there was an error
      this.scheduleNextCycle();
    }
  }

  scheduleNextCycle() {
    if (!this.keepRunning) return;

    const randomAdditionalDelay = Math.random() * MAX_RANDOM_ADDITIONAL_DELAY;
    const currentInterval = BASE_INTERVAL + randomAdditionalDelay;

    console.log(
      `Next attempt in ${(currentInterval / 1000).toFixed(2)} seconds.`
    );
    console.log("Please wait...");
    setTimeout(() => this.processGuildCycle(), currentInterval);
  }
}

// Initial start of the script
console.log("===== Guild Creation Script =====");
console.log("       Script by cyr0nix         ");
console.log("=================================");
console.log("Starting guild creation script with randomized intervals.");
console.log(
  `Base interval: ${BASE_INTERVAL / 1000}s. Max additional random delay: ${
    MAX_RANDOM_ADDITIONAL_DELAY / 1000
  }s.`
);
console.log("----------------------------------------");

const guildCreator = new GuildCreator();
guildCreator.processGuildCycle();
```

</details>

---

### Sorun Giderme

1. **Neden `Uncaught TypeError: Cannot read properties of null (reading 'createGuildFromTemplate')` hatasını alıyorum?**  
   1.1. Betiği çalıştırmadan önce manuel olarak yeni bir sunucu oluşturmanız gerekir; aksi takdirde, betik yeni bir sunucu oluşturamaz. Bu betiği tekrar çalıştırmanız gerekirse, bu adımı tekrarlamanız gerekecektir.

2. **Neden `Uncaught ReferenceError: findByProps is not defined` hatasını alıyorum?**  
   2.1. **Vencord**'u yüklemeniz ve **ConsoleShortcuts** eklentisini etkinleştirmeniz gerekiyor (**ConsoleJanitor** değil!). Bundan sonra, istemcinizi yeniden başlatın, manuel olarak bir sunucu oluşturun ve ardından betiği çalıştırın.
  
3. **Konsoldan "Hold Up!" mesajlarını nasıl kaldırabilirim?**  
   3.1. Bu tür mesajları ortadan kaldırmak için **ConsoleJanitor** eklentisini kullanabilirsiniz, ancak bu gerekli bir adım değildir.
