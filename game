import pygame

import sys

import random

import math

import time



# --- Initialization ---

pygame.init()

WIDTH, HEIGHT = 800, 600

screen = pygame.display.set_mode((WIDTH, HEIGHT))

pygame.display.set_caption("Survivor Arena - Barrels & Lasers")

clock = pygame.time.Clock()



# Fonts

font = pygame.font.SysFont("Arial", 40, bold=True)

small_font = pygame.font.SysFont("Arial", 22, bold=True)

tiny_font = pygame.font.SysFont("Arial", 18, bold=True)



# --- Global State ---

player_x, player_y = float(WIDTH // 2), float(HEIGHT // 2)

player_speed = 6.5

player_health = 100

player_max_health = 100

score = 500 



enemies = []

barrels = []

explosions = []

active_laser_beam = None

active_swipe = None



# Upgradable Stats

has_gun = False 

laser_dmg = 1

swipe_size = 90 

ability_cooldown = 0.25 

enemy_spawn_rate = 45 



# --- Combat Logic ---

last_dx, last_dy = 1, 0 

last_ability_time = 0

shake_amount = 0

iframe_duration = 0.5

last_hit_time = 0



# Game Progress

current_state = "playing"

wave, wave_timer = 1, 0

WAVE_DURATION = 3600 

BREAK_DURATION = 10  

break_start_time = 0



# --- Shop Data ---

shop_open = False

current_tab = "Offense"

tab_names = ["Offense", "Utility"]



offense_items = [

    {"name": "Laser Blaster", "cost": 500, "id": "gun", "owned": False},

    {"name": "Steel Edge (Size+)", "cost": 150, "id": "swipe_up", "owned": False},

    {"name": "Plasma Core (Dmg+)", "cost": 400, "id": "dmg_up", "owned": False},

]

utility_items = [

    {"name": "Speed Boost", "cost": 150, "id": "speed", "owned": False},

    {"name": "Armor (Heal)", "cost": 200, "id": "armor", "owned": False},

]



# --- Classes ---



class Barrel:

    def __init__(self):

        self.x = random.randint(100, WIDTH - 100)

        self.y = random.randint(100, HEIGHT - 100)

        self.size = 15



    def draw(self, surf, ox, oy):

        rect = pygame.Rect(self.x + ox - self.size, self.y + oy - self.size, self.size * 2, self.size * 2)

        pygame.draw.rect(surf, (200, 50, 50), rect) # Red barrel

        pygame.draw.rect(surf, (50, 50, 50), rect, 2) # Outline

        # Yellow hazard stripe

        pygame.draw.rect(surf, (255, 200, 0), (self.x + ox - 8, self.y + oy - 3, 16, 6))



class Explosion:

    def __init__(self, x, y):

        self.x, self.y = x, y

        self.radius = 0

        self.timer = 20

        self.max_radius = 120



    def update(self):

        self.timer -= 1

        # Grows then fades

        self.radius = (1 - (self.timer / 20)) * self.max_radius



# --- Functions ---



def spawn_enemy():

    side = random.choice(["top", "bottom", "left", "right"])

    if side == "top": ex, ey = random.randint(0, WIDTH), -30

    elif side == "bottom": ex, ey = random.randint(0, WIDTH), HEIGHT + 30

    elif side == "left": ex, ey = -30, random.randint(0, HEIGHT)

    else: ex, ey = WIDTH + 30, random.randint(0, HEIGHT)

    hp = 3.0 + (wave * 0.5)

    enemies.append({'x': ex, 'y': ey, 'color': (255, 60, 60), 'speed': 2.2, 'hp': hp, 'max_hp': hp, 'size': 12, 'flash': 0})



def handle_shop_purchase(item):

    global score, player_speed, player_health, player_max_health, has_gun, laser_dmg, swipe_size

    if not item["owned"] and score >= item["cost"]:

        score -= item["cost"]

        uid = item["id"]

        if uid == "gun": has_gun = True; item["owned"] = True 

        elif uid == "swipe_up": swipe_size += 25

        elif uid == "dmg_up": laser_dmg += 1

        elif uid == "speed": player_speed += 0.5

        elif uid == "armor": player_health = min(player_max_health, player_health + 50)

        

        if uid != "gun": item["cost"] = int(item["cost"] * 1.6)



def trigger_explosion(x, y):

    global shake_amount

    explosions.append(Explosion(x, y))

    shake_amount = 20

    # Damage nearby enemies

    for e in enemies:

        if math.hypot(e['x'] - x, e['y'] - y) < 120:

            e['hp'] -= 5

            e['flash'] = 10



def fire_laser(start_x, start_y, angle):

    global active_laser_beam

    length = 1000

    end_x = start_x + math.cos(angle) * length

    end_y = start_y + math.sin(angle) * length

    active_laser_beam = {"start": (start_x, start_y), "end": (end_x, end_y), "timer": 6}



    # Hit enemies

    for e in enemies:

        dist = abs((end_y - start_y)*e['x'] - (end_x - start_x)*e['y'] + end_x*start_y - end_y*start_x) / math.hypot(end_y - start_y, end_x - start_x)

        if dist < e['size'] + 15:

            e['hp'] -= laser_dmg

            e['flash'] = 5

    

    # Hit Barrels

    for b in barrels[:]:

        dist = abs((end_y - start_y)*b.x - (end_x - start_x)*b.y + end_x*start_y - end_y*start_x) / math.hypot(end_y - start_y, end_x - start_x)

        if dist < b.size + 10:

            trigger_explosion(b.x, b.y)

            barrels.remove(b)



def draw_better_swipe(screen, px, py, ox, oy, swipe_data, s_size):

    progress = swipe_data["timer"] / swipe_data["max_timer"]

    base_angle = swipe_data["angle"]

    sweep_range = 1.8

    current_sweep = (progress * sweep_range) - (sweep_range / 2)

    s_angle = base_angle - current_sweep

    swipe_surf = pygame.Surface((s_size * 2.5, s_size * 2.5), pygame.SRCALPHA)

    center = (s_size * 1.25)

    pts = []

    arc_span = 0.6

    for step in range(11):

        a = s_angle + (-arc_span / 2) + (step * (arc_span / 10))

        pts.append((center + math.cos(a)*s_size, center + math.sin(a)*s_size))

    pts.append((center, center))

    if len(pts) >= 3:

        pygame.draw.polygon(swipe_surf, (0, 200, 255, int(200 * progress)), pts)

        pygame.draw.polygon(swipe_surf, (255, 255, 255, int(255 * progress)), pts, 2)

    screen.blit(swipe_surf, (px + 10 + ox - center, py + 10 + oy - center))



# Initial barrels

barrels = [Barrel() for _ in range(random.randint(3, 6))]



# --- Main Loop ---

while True:

    current_time = time.time()

    mouse_pos = pygame.mouse.get_pos()

    

    for event in pygame.event.get():

        if event.type == pygame.QUIT: pygame.quit(); sys.exit()

        if event.type == pygame.KEYDOWN:

            if event.key == pygame.K_e: shop_open = not shop_open

            if not shop_open and event.key == pygame.K_f:

                if current_time - last_ability_time >= ability_cooldown:

                    last_ability_time = current_time

                    shoot_angle = math.atan2(last_dy, last_dx)

                    

                    if has_gun:

                        fire_laser(player_x + 10, player_y + 10, shoot_angle)

                    else:

                        active_swipe = {"timer": 8, "max_timer": 8, "angle": shoot_angle}

                        # Check enemies

                        for e in enemies:

                            if math.hypot(player_x + 10 - e['x'], player_y + 10 - e['y']) < swipe_size + 20:

                                e['hp'] -= 3; e['flash'] = 5 

                        # Check barrels

                        for b in barrels[:]:

                            if math.hypot(player_x + 10 - b.x, player_y + 10 - b.y) < swipe_size + 20:

                                trigger_explosion(b.x, b.y)

                                barrels.remove(b)



        if event.type == pygame.MOUSEBUTTONDOWN and shop_open:

            for i, name in enumerate(tab_names):

                if pygame.Rect(200 + (i * 100), 140, 100, 30).collidepoint(mouse_pos): current_tab = name

            active_list = {"Offense": offense_items, "Utility": utility_items}[current_tab]

            for i, item in enumerate(active_list):

                if pygame.Rect(230, 190 + (i * 50), 340, 40).collidepoint(mouse_pos): 

                    handle_shop_purchase(item)



    if not shop_open:

        keys = pygame.key.get_pressed()

        dx, dy = keys[pygame.K_d] - keys[pygame.K_a], keys[pygame.K_s] - keys[pygame.K_w]

        if dx != 0 or dy != 0: last_dx, last_dy = dx, dy

        player_x = max(0, min(player_x + dx * player_speed, WIDTH - 20))

        player_y = max(0, min(player_y + dy * player_speed, HEIGHT - 20))



        if current_state == "playing":

            wave_timer += 1

            if wave_timer % enemy_spawn_rate == 0: spawn_enemy()

            

            if wave_timer >= WAVE_DURATION:

                current_state, break_start_time, score = "break", current_time, score + 300

                wave_timer = 0; enemies.clear()

                barrels = [Barrel() for _ in range(random.randint(3, 6))]



            for e in enemies[:]:

                dist = math.hypot(player_x + 10 - e['x'], player_y + 10 - e['y'])

                if e['hp'] <= 0: enemies.remove(e); score += 15; continue 

                if dist > 0:

                    e['x'] += (player_x + 10 - e['x']) / dist * e['speed']

                    e['y'] += (player_y + 10 - e['y']) / dist * e['speed']

                if dist < e['size'] + 10 and current_time - last_hit_time >= iframe_duration:

                    player_health -= 10; last_hit_time, shake_amount = current_time, 10

                    if player_health <= 0: pygame.quit(); sys.exit()

        

        elif current_state == "break":

            if current_time - break_start_time >= BREAK_DURATION:

                current_state, wave = "playing", wave + 1

        

        for ex in explosions[:]:

            ex.update()

            if ex.timer <= 0: explosions.remove(ex)



        if active_swipe: active_swipe["timer"] -= 1

        if active_swipe and active_swipe["timer"] <= 0: active_swipe = None

        if active_laser_beam: 

            active_laser_beam["timer"] -= 1

            if active_laser_beam["timer"] <= 0: active_laser_beam = None



    # --- Drawing ---

    screen.fill((5, 5, 15))

    ox, oy = (random.randint(-shake_amount, shake_amount), random.randint(-shake_amount, shake_amount)) if shake_amount > 0 else (0,0)

    if shake_amount > 0: shake_amount -= 1



    for b in barrels: b.draw(screen, ox, oy)



    for e in enemies: 

        e_color = (255, 255, 255) if e['flash'] > 0 else e['color']

        if e['flash'] > 0: e['flash'] -= 1

        pygame.draw.circle(screen, e_color, (int(e['x'] + ox), int(e['y'] + oy)), e['size'])

        bx, by = e['x'] + ox - e['size'], e['y'] + oy - e['size'] - 12

        pygame.draw.rect(screen, (80, 0, 0), (bx, by, e['size']*2, 5))

        pygame.draw.rect(screen, (0, 255, 100), (bx, by, int(e['size']*2 * (e['hp']/e['max_hp'])), 5))



    for ex in explosions:

        alpha = int(255 * (ex.timer / 20))

        pygame.draw.circle(screen, (255, 150, 0), (int(ex.x + ox), int(ex.y + oy)), int(ex.radius), 2)

        s = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)

        pygame.draw.circle(s, (255, 100, 0, alpha // 2), (int(ex.x + ox), int(ex.y + oy)), int(ex.radius))

        screen.blit(s, (0,0))



    if active_laser_beam:

        pygame.draw.line(screen, (0, 150, 255), active_laser_beam["start"], active_laser_beam["end"], 6)

        pygame.draw.line(screen, (255, 255, 255), active_laser_beam["start"], active_laser_beam["end"], 2)



    if active_swipe: draw_better_swipe(screen, player_x, player_y, ox, oy, active_swipe, swipe_size)

    pygame.draw.rect(screen, (0, 200, 255), (int(player_x + ox), int(player_y + oy), 20, 20))

    

    # UI

    if current_state == "playing":

        rem = (WAVE_DURATION - wave_timer)//60

        timer_text = f"WAVE {wave}: {rem//60:02d}:{rem%60:02d}"

    else:

        timer_text = f"BREAK: {int(BREAK_DURATION - (current_time - break_start_time))}s"

    

    t_surf = font.render(timer_text, True, (255,255,255))

    screen.blit(t_surf, (WIDTH // 2 - t_surf.get_width() // 2, 20))

    screen.blit(small_font.render(f"GOLD: {score}", True, (255, 215, 0)), (15, 15))

    pygame.draw.rect(screen, (100, 0, 0), (WIDTH - 170, 15, 150, 15))

    pygame.draw.rect(screen, (0, 255, 100), (WIDTH - 170, 15, int(150 * (player_health/player_max_health)), 15))



    if shop_open:

        overlay = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA); overlay.fill((0, 0, 0, 200)); screen.blit(overlay, (0,0))

        pygame.draw.rect(screen, (30, 30, 40), (180, 90, 440, 420))

        for i, name in enumerate(tab_names):

            tc = (100, 100, 255) if current_tab == name else (60, 60, 70)

            pygame.draw.rect(screen, tc, (200 + (i*100), 140, 95, 30))

            screen.blit(tiny_font.render(name, True, (255,255,255)), (210+(i*100), 145))

        active_list = {"Offense": offense_items, "Utility": utility_items}[current_tab]

        for i, item in enumerate(active_list):

            r = pygame.Rect(230, 190 + (i * 50), 340, 40)

            pygame.draw.rect(screen, (50, 50, 60), r)

            status = "SOLD" if item["owned"] else f"${item['cost']}"

            color = (100, 255, 100) if item["owned"] else (255, 215, 0) if score >= item["cost"] else (200, 50, 50)

            screen.blit(small_font.render(f"{item['name']} - {status}", True, color), (240, 200+(i*50)))



    pygame.display.flip()

    clock.tick(60)
