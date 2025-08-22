# levne-letenkyimport requests
from feedgen.feed import FeedGenerator
import os

API_KEY = os.environ["KIWI_API_KEY"]

# seznam destinací – můžeš si upravit
DESTINACE = [
    ("PRG", "LON", "01/09/2025", "30/09/2025", 50),  # Praha → Londýn
    ("PRG", "PAR", "01/09/2025", "30/09/2025", 60),  # Praha → Paříž
    ("PRG", "ROM", "01/09/2025", "30/09/2025", 70),  # Praha → Řím
]

def najdi_lety(z, kam, datum_od, datum_do, max_cena):
    url = "https://tequila-api.kiwi.com/v2/search"
    headers = {"apikey": API_KEY}
    params = {
        "fly_from": z,
        "fly_to": kam,
        "date_from": datum_od,
        "date_to": datum_do,
        "curr": "EUR",
        "price_to": max_cena,
        "limit": 10,
        "sort": "price"
    }
    r = requests.get(url, headers=headers, params=params)
    data = r.json()
    return data.get("data", [])

def vytvor_rss(vystup="letenky.xml"):
    fg = FeedGenerator()
    fg.title("Levné letenky Evropa")
    fg.link(href="http://example.com", rel="self")
    fg.description("Automaticky generované tipy na levné lety")

    for (z, kam, od, do, cena) in DESTINACE:
        lety = najdi_lety(z, kam, od, do, cena)
        for let in lety:
            fe = fg.add_entry()
            cena = let["price"]
            z = let["cityFrom"]
            kam = let["cityTo"]
            datum = let["local_departure"][:10]
            url = let["deep_link"]

            fe.title(f"{z} → {kam} za {cena} EUR ({datum})")
            fe.link(href=url)
            fe.description(f"Létá {z} → {kam}, cena {cena} EUR, datum {datum}")

    fg.rss_file(vystup)
    print(f"✅ RSS feed uložen do {vystup}")

if __name__ == "__main__":
    vytvor_rss()
