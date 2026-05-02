import random
import time
import os

class Player:
    def __init__(self, name):
        self.name = name
        self.health = 100
        self.max_health = 100
        self.hunger = 100
        self.thirst = 100
        self.inventory = []
        self.equipped_armor = None
        self.armor_defense = 0
        self.day = 1
        
    def is_alive(self):
        return self.health > 0 and self.hunger > 0 and self.thirst > 0
        
    def show_stats(self):
        print("\n" + "="*40)
        print(f"👤 {self.name} | Gün: {self.day}")
        print(f"❤️ Can: {self.health}/{self.max_health}")
        health_bar = "█" * int(self.health/10) + "░" * (10 - int(self.health/10))
        print(f"   [{health_bar}]")
        print(f"🍖 Açlık: {self.hunger}/100")
        hunger_bar = "█" * int(self.hunger/10) + "░" * (10 - int(self.hunger/10))
        print(f"   [{hunger_bar}]")
        print(f"💧 Susuzluk: {self.thirst}/100")
        thirst_bar = "█" * int(self.thirst/10) + "░" * (10 - int(self.thirst/10))
        print(f"   [{thirst_bar}]")
        if self.equipped_armor:
            print(f"🛡️ Zırh: {self.equipped_armor} (+{self.armor_defense})")
        else:
            print(f"🛡️ Zırh: Yok")
        print("="*40)
            def show_inventory(self):
        print("\n" + "="*40)
        print("🎒 ENVANTER")
        if not self.inventory:
            print("   Boş!")
        else:
            for item in self.inventory:
                print(f"   • {item}")
        print("="*40)
        
    def add_item(self, item):
        self.inventory.append(item)
        print(f"📦 {item} eklendi!")
        
    def remove_item(self, item):
        if item in self.inventory:
            self.inventory.remove(item)
            return True
        return False
        
    def take_damage(self, damage):
        actual_damage = max(1, damage - self.armor_defense)
        self.health -= actual_damage
        print(f"💔 {actual_damage} hasar! Can: {self.health}")
        
    def use_item(self, item):
        if item == "Meyve" and item in self.inventory:
            self.hunger = min(100, self.hunger + 20)
            self.thirst = min(100, self.thirst + 10)
            self.remove_item(item)
            print("🍎 Meyve yendi! Açlık+20 Susuzluk+10")
            return True
        elif item == "Su Şişesi" and item in self.inventory:
            self.thirst = min(100, self.thirst + 40)
            self.remove_item(item)
            print("💧 Su içildi! Susuzluk+40")
            return True
        elif item == "Et" and item in self.inventory:
            self.hunger = min(100, self.hunger + 35)
            self.remove_item(item)
            print("🍖 Et yendi! Açlık+35")
            return True
        elif "Deri Zırh" in item and item in self.inventory:
            self.equipped_armor = item
            if "Göğüslük" in item:
                self.armor_defense = 5
            elif "Pantolon" in item:
                self.armor_defense = 3
            elif "Başlık" in item:
                self.armor_defense = 2
            print(f"✨ {item} giyildi! Savunma+{self.armor_defense}")
            return True
        return False
        crafting_recipes = {
    "Halat": {"Odun": 2, "Bitki Lifi": 3},
    "Taş Balta": {"Odun": 3, "Taş": 2, "Halat": 1},
    "Taş Mızrak": {"Odun": 4, "Taş": 3, "Halat": 1},
    "Deri Zırh (Göğüslük)": {"Deri": 5, "Halat": 2},
    "Deri Zırh (Pantolon)": {"Deri": 3, "Halat": 1},
    "Deri Zırh (Başlık)": {"Deri": 2, "Halat": 1},
    "Kürk Zırh": {"Kürk": 6, "Halat": 3},
    "Ateş": {"Odun": 5, "Taş": 2},
    "Barınak": {"Odun": 10, "Taş": 5, "Halat": 3}
}

class World:
    def __init__(self):
        self.locations = {
            "orman": ["ağaç", "yabani meyve", "kurt", "ayı", "bitki", "tavşan"],
            "mağara": ["taş", "eski sandık", "yabani meyve", "kurt", "taş"],
            "nehir": ["su", "bitki", "balık", "tavşan", "taş", "yabani meyve"],
            "harabe": ["eski sandık", "taş", "kurt", "bitki", "eski sandık"]
        }
        
    def gather(self, location):
        if location not in self.locations:
            return None, None
        possible_items = {
            "ağaç": "Odun", "taş": "Taş", "yabani meyve": "Meyve",
            "bitki": "Bitki Lifi", "su": "Su Şişesi", "balık": "Et"
        }
        available = []
        for item in self.locations[location]:
            if item in possible_items:
                available.append(possible_items[item])
        if not available:
            return None, None
        found = random.choice(available)
        if found == "Odun":
            return found, random.randint(1, 3)
        elif found == "Taş":
            return found, random.randint(1, 2)
        elif found == "Bitki Lifi":
            return found, random.randint(1, 4)
        else:
            return found, 1
                def hunt(self, location):
        animals = {
            "orman": ["tavşan", "kurt", "ayı"],
            "mağara": ["kurt"],
            "nehir": ["balık", "tavşan"],
            "harabe": ["kurt"]
        }
        if location not in animals:
            return None
        animal = random.choice(animals[location])
        if animal == "tavşan":
            return {"animal": animal, "success_chance": 0.8, "drops": ["Et", "Deri"], "count": random.randint(1,2), "damage": 5}
        elif animal == "balık":
            return {"animal": animal, "success_chance": 0.7, "drops": ["Et"], "count": 1, "damage": 3}
        elif animal == "kurt":
            return {"animal": animal, "success_chance": 0.5, "drops": ["Et", "Deri", "Kürk"], "count": random.randint(1,2), "damage": 20}
        elif animal == "ayı":
            return {"animal": animal, "success_chance": 0.3, "drops": ["Et", "Deri", "Kürk"], "count": random.randint(2,4), "damage": 35}
        
    def explore_ruins(self):
        loot_table = [
            ("Odun", random.randint(2,5), 80), ("Taş", random.randint(1,3), 70),
            ("Meyve", 1, 60), ("Su Şişesi", 1, 50), ("Et", 1, 40),
            ("Deri", random.randint(1,2), 35), ("Halat", 1, 30)
        ]
        found_items = []
        for item, amount, chance in loot_table:
            if random.randint(1, 100) <= chance:
                found_items.append((item, amount))
        danger = random.choice([None, "kurt", "tuzak", "çöküş"])
        return found_items, danger
        def clear_screen():
    os.system('cls' if os.name == 'nt' else 'clear')

def craft_item(player):
    print("\n" + "="*40)
    print("🔨 CRAFTING")
    recipes = list(crafting_recipes.keys())
    for i, recipe in enumerate(recipes, 1):
        print(f"   {i}. {recipe}")
    print("   0. Geri")
    choice = input("\nSeçim: ").strip()
    if not choice.isdigit() or int(choice) == 0:
        return
    choice = int(choice)
    if 1 <= choice <= len(recipes):
        recipe_name = recipes[choice-1]
        reqs = crafting_recipes[recipe_name]
        can_craft = True
        for req_item, req_amount in reqs.items():
            if player.inventory.count(req_item) < req_amount:
                print(f"❌ Yetersiz: {req_item}")
                can_craft = False
        if can_craft:
            for req_item, req_amount in reqs.items():
                for _ in range(req_amount):
                    player.remove_item(req_item)
            if recipe_name == "Ateş":
                print("🔥 Ateş yakıldı!")
            elif recipe_name == "Barınak":
                print("🏠 Barınak yapıldı!")
            else:
                player.add_item(recipe_name)
                print(f"✅ {recipe_name} üretildi!")
        else:
            print("❌ Malzeme yetersiz!")
    input("\nEnter'a bas...")

def main():
    clear_screen()
    print("="*50)
    print("      🌲 HAYATTA KALMA OYUNU 🌲")
    print("="*50)
    name = input("\nKarakter adı: ").strip()
    if not name:
        name = "Hayatta Kalan"
    player = Player(name)
    world = World()
    print(f"\nHoş geldin, {name}!")
    input("\nBaşlamak için Enter'a bas...")
        while player.is_alive():
        clear_screen()
        player.show_stats()
        
        if player.day > 1:
            player.hunger -= random.randint(5, 10)
            player.thirst -= random.randint(8, 15)
            player.hunger = max(0, player.hunger)
            player.thirst = max(0, player.thirst)
        
        print("\n📍 NEREYE?")
        print("   1. 🌲 Orman")
        print("   2. ⛰️ Mağara")
        print("   3. 💧 Nehir")
        print("   4. 🏚️ Harabe")
        print("   5. 🎒 Envanter")
        print("   6. 🔨 Crafting")
        print("   7. 📦 Eşya kullan")
        print("   8. 😴 Günü bitir")
        
        choice = input("\nSeçim (1-8): ").strip()
        
        if choice == "1" or choice == "2" or choice == "3" or choice == "4":
            loc_map = {"1":"orman","2":"mağara","3":"nehir","4":"harabe"}
            location = loc_map[choice]
            print(f"\n📍 {location.upper()}")
            print("   a) Toplama")
            print("   b) Avlanma")
            if location == "harabe":
                print("   c) Harabeyi keşfet")
            action = input("\nSeçim (a/b/c): ").strip().lower()
            
            if action == "a":
                item, amount = world.gather(location)
                if item:
                    for _ in range(amount):
                        player.add_item(item)
                    print(f"✅ {amount} {item} toplandı!")
                else:
                    print("❌ Toplanacak şey yok!")
                    
            elif action == "b":
                hunt = world.hunt(location)
                if hunt:
                    print(f"\n🐺 {hunt['animal']} görüldü!")
                    time.sleep(1)
                    if random.random() < hunt['success_chance']:
                        print(f"⚔️ Kazandın!")
                        for drop in hunt['drops']:
                            for _ in range(hunt['count']):
                                player.add_item(drop)
                            print(f"📦 {hunt['count']} {drop} kazanıldı!")
                    else:
                        print(f"💀 Yaralandın!")
                        player.take_damage(hunt['damage'])
                else:
                    print("❌ Av yok!")
                    
            elif action == "c" and location == "harabe":
                items, danger = world.explore_ruins()
                if danger:
                    print(f"\n⚠️ {danger.upper()}!")
                    if danger == "kurt":
                        player.take_damage(20)
                    elif danger == "tuzak":
                        player.take_damage(15)
                    elif danger == "çöküş":
                        player.take_damage(10)
                if items:
                    for item, amount in items:
                        for _ in range(amount):
                            player.add_item(item)
                        print(f"📦 {item} x{amount} bulundu!")
                else:
                    print("❌ Hiçbir şey bulunamadı!")
                            elif choice == "5":
            player.show_inventory()
            input("\nEnter'a bas...")
            
        elif choice == "6":
            craft_item(player)
            
        elif choice == "7":
            player.show_inventory()
            item = input("\nKullanılacak eşya: ").strip()
            player.use_item(item)
            input("\nEnter'a bas...")
            
        elif choice == "8":
            player.day += 1
            print(f"\n🌙 Gün {player.day-1} bitti!")
            time.sleep(1)
            
        if player.hunger <= 0:
            print("\n💀 Açlıktan ölüyorsun!")
            player.take_damage(10)
        if player.thirst <= 0:
            print("\n💀 Susuzluktan ölüyorsun!")
            player.take_damage(10)
            
        time.sleep(1)
        
    clear_screen()
    print("="*40)
    print("💀 GAME OVER 💀")
    print("="*40)
    print(f"\n{player.name} {player.day} gün hayatta kaldı!")
    input("\nÇıkmak için Enter'a bas...")

if __name__ == "__main__":
    main()
