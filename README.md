import json
import os
import random
import time
import urllib.request

TOKEN = "8541698879:AAEkXgnsKhDDVjIzUdJ0qj9juBbI_JqvZ6w"
URL = f"https://api.telegram.org/bot{TOKEN}/"

ADMIN_SECRET_KOD = "vip_arsi1245"
DATA_FILE = "users_data.json"
TIKTOK_USERNAME = "arsi61122"


def data_okap_al():
  if os.path.exists(DATA_FILE):
    try:
      with open(DATA_FILE, "r") as f:
        return json.load(f)
    except:
      return {}
  return {}


def data_yaz(data):
  with open(DATA_FILE, "w") as f:
    json.dump(data, f)


def mugt_kod_tap():
  san = random.randint(1000, 9999)
  serverler = ["de", "nl", "us", "fr"]
  secilen = random.choice(serverler)
  return f"vless://mugt-1gun-{secilen}-{san}@server.net:443?encryption=none"


def vip_kod_tap():
  san = random.randint(10000, 99999)
  return f"vless://VIP-1aylyk-{san}@vip-server.net:443?encryption=none"


def mesaj_iber(chat_id, text, reply_markup=None):
  data = {"chat_id": chat_id, "text": text, "parse_mode": "Markdown"}
  if reply_markup:
    data["reply_markup"] = reply_markup

  req = urllib.request.Request(
      URL + "sendMessage",
      data=json.dumps(data).encode("utf-8"),
      headers={"Content-Type": "application/json"},
  )
  try:
    urllib.request.urlopen(req)
  except Exception as e:
    print("Ýalňyşlyk:", e)


def baslangic_klawiatura():
  return {
      "keyboard": [
          [{"text": "✅ TikTok-a agza boldum (Barlamak)"}],
          [{"text": "⭐ VIP kod almak (150 manat - 1 aýlyk)"}],
          [{"text": "📱 Nomerim arkaly kod al", "request_contact": True}],
      ],
      "resize_keyboard": True,
      "one_time_keyboard": False,
  }


def tiktok_silka_klawiatura():
  return {
      "inline_keyboard": [[
          {
              "text": "🔗 TikTok Sahypasyna Geçmek (@" + TIKTOK_USERNAME + ")",
              "url": f"https://www.tiktok.com/@{TIKTOK_USERNAME}",
          }
      ]]
  }


print("Bot ähli funksiýalar bilen doly işe girizildi...")
offset = 0

while True:
  try:
    req = urllib.request.urlopen(f"{URL}getUpdates?offset={offset}&timeout=30")
    response = json.loads(req.read().decode("utf-8"))

    for result in response.get("result", []):
      offset = result["update_id"] + 1

      if "message" in result:
        mesaj = result["message"]
        chat_id = str(mesaj["chat"]["id"])
        user_db = data_okap_al()

        if chat_id not in user_db:
          user_db[chat_id] = {
              "last_tiktok": 0,
              "subscribed": True,
              "waiting_for_phone": False,
          }

        if "contact" in mesaj:
          telefon = mesaj["contact"].get("phone_number")
          kod = mugt_kod_tap()
          mesaj_iber(
              chat_id,
              f"Sag boluň! Nomeriňiz kabul edildi: +{telefon}\n\nSiziň"
              f" mugt VPN koduňyz:\n`{kod}`",
          )

        elif "text" in mesaj:
          text = mesaj["text"]
          simdi = time.time()

          # Admin kody / VIP alyş kody
          if text == ADMIN_SECRET_KOD or text.lower() == "/kod":
            vip_kod = vip_kod_tap()
            mesaj_iber(
                chat_id,
                f"👑 **Admin tassyklady!**\n\nSiziň çäklendirmesiz 1 aýlyk VIP"
                f" koduňyz:\n`{vip_kod}`\n\nGowy ulanmalar! 🔥",
            )

          elif text == "/start":
            user_db[chat_id]["waiting_for_phone"] = False
            data_yaz(user_db)
            mesaj_iber(
                chat_id,
                f"Salam! Turkmenistan VPN botuna hoş geldiňiz.\n\n"
                f"• Ilki bilen aşakdaky düwmeden TikTok sahypamyza"
                f" (**@{TIKTOK_USERNAME}**) abuna boluň.\n"
                f"• VIP kod almak isleýänler 150 manat tölemeli.",
                reply_markup=tiktok_silka_klawiatura(),
            )
            mesaj_iber(
                chat_id,
                "Aşakdaky düwmeleri ulanyň:",
                reply_markup=baslangic_klawiatura(),
            )

          elif text == "✅ TikTok-a agza boldum (Barlamak)":
            sonky_wagt = user_db[chat_id].get("last_tiktok", 0)
            bir_hepde = 7 * 24 * 60 * 60

            if simdi - sonky_wagt < bir_hepde:
              galan_wagt = bir_hepde - (simdi - sonky_wagt)
              gun = int(galan_wagt // (24 * 60 * 60))
              sagat = int((galan_wagt % (24 * 60 * 60)) // 3600)
              mesaj_iber(
                  chat_id,
                  f"⚠️ 1 hepdelik möhlet dolmady!\nTäzeden kod almak üçin"
                  f" ýene **{gun} gün {sagat} sagat** garaşmaly.",
              )
            else:
              kod = mugt_kod_tap()
              user_db[chat_id]["last_tiktok"] = simdi
              data_yaz(user_db)
              mesaj_iber(
                  chat_id,
                  f"Gutlaýaryn! TikTok agzalygy tassyklandy.\n\nSiziň 1"
                  f" günlük mugt VPN koduňyz:\n`{kod}`\n\nGowy ulanmalar!"
                  f" 😉",
              )

          elif text == "⭐ VIP kod almak (150 manat - 1 aýlyk)":
            user_db[chat_id]["waiting_for_phone"] = True
            data_yaz(user_db)
            mesaj_iber(
                chat_id,
                "Hormatly ulanyjy, siz bu 1 aýlyk Vless kody aljak"
                " bolsanyz, şul nomera **0804** arkaly puluňyzy oklap"
                " iberiň.\n\nZähmet bolmasa, töleg eden telefon"
                " nomeriňizi aşakda ýazyň (mysal üçin:"
                " `+99361234567`):",
            )

          else:
            if user_db[chat_id].get("waiting_for_phone", False):
              telefon_nom = text
              user_db[chat_id]["waiting_for_phone"] = False
              data_yaz(user_db)

              mesaj_iber(
                  chat_id,
                  f"✅ Nomeriňiz kabul edildi: `{telefon_nom}`\n\nAdmin"
                  f" tölegiňizi barlansoň, size VIP kody iberer. Garaşmagyňyzy"
                  f" haýyş edýäris!",
              )

              # Admine awtomatiki habar gitmegi
              mesaj_iber(
                  chat_id,
                  f"🔔 **Täze VIP Töleg Soragy!**\n\nUlanyjy 1 aýlyk vless"
                  f" kod aljak bolýar.\nTöleg eden nomeri:"
                  f" `{telefon_nom}`\nTelegram ID: `{chat_id}`\n\nKody ugratmak"
                  f" üçin `/kod` ýazyň.",
              )
            else:
              mesaj_iber(chat_id, "Zahmat bolmasa, aşakdaky düwmeleri ulanyň.")
  except Exception as e:
    print("Ýalňyşlyk:", e)
