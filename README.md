import pygame
import random
import sys
import math

# ─────────────────────────────────────────
#  INIT
# ─────────────────────────────────────────
pygame.init()
pygame.font.init()

WIDTH, HEIGHT = 480, 700
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Mapel Dodge — by Abbilefa")
clock = pygame.time.Clock()
FPS = 60

# ─────────────────────────────────────────
#  COLORS  (dark navy + blue theme)
# ─────────────────────────────────────────
C_BG          = (8,  14,  30)       # very dark navy
C_BG2         = (12, 22,  50)       # slightly lighter navy
C_PANEL       = (15, 28,  65)       # panel bg
C_BORDER      = (30, 80, 180)       # blue border
C_ACCENT      = (60, 140, 255)      # light blue accent
C_ACCENT2     = (100, 200, 255)     # cyan-ish accent
C_WHITE       = (220, 235, 255)     # soft white/blue-white
C_DIM         = (80,  110, 160)     # dimmed text
C_GOOD        = (50,  220, 130)     # green for target
C_GOOD_DIM    = (20,  80,  50)      # dark green
C_DANGER      = (255,  70,  80)     # red for danger/wrong
C_DANGER_DIM  = (80,   20,  25)     # dark red
C_GOLD        = (255, 200,  50)     # gold for score/level
C_HEART       = (255,  80, 100)     # heart color
C_SHADOW      = (0,    0,   0, 160)

# ─────────────────────────────────────────
#  MAPEL DATA
# ─────────────────────────────────────────
MAPEL_LIST = [
    ("MATEMATIKA",    (60,  130, 255)),
    ("FISIKA",        (100, 180, 255)),
    ("BIOLOGI",       (50,  220, 130)),
    ("KIMIA",         (180, 100, 255)),
    ("GEOGRAFI",      (255, 170,  60)),
    ("SOSIOLOGI",     (255, 220,  60)),
    ("EKONOMI",       (60,  210, 180)),
    ("SEJARAH",       (255, 130,  60)),
    ("PKN",           (255,  80, 100)),
    ("SENI RUPA",     (255, 120, 200)),
    ("PEND. AGAMA",   (255, 200,  80)),
    ("INFORMATIKA",   (80,  220, 255)),
    ("OLAHRAGA",      (130, 255, 130)),
]

# ─────────────────────────────────────────
#  FONTS
# ─────────────────────────────────────────
try:
    FONT_TITLE  = pygame.font.SysFont("impact", 54, bold=True)
    FONT_BIG    = pygame.font.SysFont("impact", 36)
    FONT_MED    = pygame.font.SysFont("consolas", 22, bold=True)
    FONT_SM     = pygame.font.SysFont("consolas", 16)
    FONT_MAPEL  = pygame.font.SysFont("consolas", 14, bold=True)
    FONT_TINY   = pygame.font.SysFont("consolas", 13)
except:
    FONT_TITLE  = pygame.font.SysFont(None, 54)
    FONT_BIG    = pygame.font.SysFont(None, 36)
    FONT_MED    = pygame.font.SysFont(None, 22)
    FONT_SM     = pygame.font.SysFont(None, 16)
    FONT_MAPEL  = pygame.font.SysFont(None, 14)
    FONT_TINY   = pygame.font.SysFont(None, 13)

# ─────────────────────────────────────────
#  HELPERS
# ─────────────────────────────────────────
def draw_rounded_rect(surf, color, rect, radius=10, border=0, border_color=None):
    pygame.draw.rect(surf, color, rect, border_radius=radius)
    if border and border_color:
        pygame.draw.rect(surf, border_color, rect, border, border_radius=radius)

def draw_text_centered(surf, text, font, color, cx, cy):
    s = font.render(text, True, color)
    surf.blit(s, (cx - s.get_width() // 2, cy - s.get_height() // 2))

def draw_text(surf, text, font, color, x, y):
    s = font.render(text, True, color)
    surf.blit(s, (x, y))

def lerp_color(c1, c2, t):
    return tuple(int(c1[i] + (c2[i] - c1[i]) * t) for i in range(3))

# ─────────────────────────────────────────
#  STAR BACKGROUND
# ─────────────────────────────────────────
stars = [(random.randint(0, WIDTH), random.randint(0, HEIGHT),
          random.uniform(0.3, 1.5), random.uniform(0, 2 * math.pi))
         for _ in range(90)]

def draw_stars(tick):
    for (x, y, size, phase) in stars:
        alpha = 0.5 + 0.5 * math.sin(tick * 0.02 + phase)
        brightness = int(60 + 120 * alpha)
        color = (brightness // 3, brightness // 2, brightness)
        pygame.draw.circle(screen, color, (int(x), int(y)), max(1, int(size)))

# ─────────────────────────────────────────
#  PARTICLE SYSTEM
# ─────────────────────────────────────────
particles = []

def spawn_particles(x, y, color, count=12):
    for _ in range(count):
        angle = random.uniform(0, 2 * math.pi)
        speed = random.uniform(2, 6)
        particles.append({
            "x": x, "y": y,
            "vx": math.cos(angle) * speed,
            "vy": math.sin(angle) * speed,
            "life": 1.0,
            "color": color,
            "size": random.uniform(3, 7)
        })

def update_draw_particles():
    dead = []
    for p in particles:
        p["x"] += p["vx"]
        p["y"] += p["vy"]
        p["vy"] += 0.15
        p["life"] -= 0.04
        if p["life"] <= 0:
            dead.append(p)
            continue
        alpha = p["life"]
        c = tuple(int(p["color"][i] * alpha) for i in range(3))
        sz = max(1, int(p["size"] * alpha))
        pygame.draw.circle(screen, c, (int(p["x"]), int(p["y"])), sz)
    for p in dead:
        particles.remove(p)

# ─────────────────────────────────────────
#  BLOCK CLASS
# ─────────────────────────────────────────
BLOCK_W, BLOCK_H = 90, 46

class Block:
    def __init__(self, speed, is_target, mapel_idx):
        self.w = BLOCK_W
        self.h = BLOCK_H
        self.x = random.randint(20, WIDTH - 20 - self.w)
        self.y = -self.h - random.randint(0, 80)
        self.speed = speed
        self.is_target = is_target
        self.mapel_name, self.base_color = MAPEL_LIST[mapel_idx]
        self.alive = True
        self.flash = 0  # flash timer on hit
        self.wobble = random.uniform(0, math.pi * 2)

    def update(self, tick):
        self.wobble += 0.05
        self.x += math.sin(self.wobble) * 0.4
        self.y += self.speed
        if self.y > HEIGHT + self.h:
            self.alive = False

    def draw(self, tick):
        rx, ry = int(self.x), int(self.y)
        rect = pygame.Rect(rx, ry, self.w, self.h)

        if self.is_target:
            # Glowing green outline
            glow_rect = rect.inflate(6, 6)
            pulse = 0.5 + 0.5 * math.sin(tick * 0.1)
            glow_color = lerp_color(C_GOOD_DIM, C_GOOD, pulse)
            draw_rounded_rect(screen, glow_color, glow_rect, radius=12)
            draw_rounded_rect(screen, (15, 50, 30), rect, radius=9)
            draw_rounded_rect(screen, (0, 0, 0, 0), rect, radius=9,
                              border=2, border_color=C_GOOD)
        else:
            shadow_rect = rect.move(3, 4)
            draw_rounded_rect(screen, (0, 0, 0), shadow_rect, radius=9)
            bg = lerp_color(C_PANEL, self.base_color, 0.18)
            draw_rounded_rect(screen, bg, rect, radius=9)
            draw_rounded_rect(screen, (0, 0, 0, 0), rect, radius=9,
                              border=2, border_color=self.base_color)

        # Label
        label_color = C_GOOD if self.is_target else self.base_color
        lines = self.mapel_name.split(" ")
        if len(lines) == 1:
            draw_text_centered(screen, lines[0], FONT_MAPEL, label_color,
                               rx + self.w // 2, ry + self.h // 2)
        else:
            draw_text_centered(screen, lines[0], FONT_MAPEL, label_color,
                               rx + self.w // 2, ry + self.h // 2 - 9)
            draw_text_centered(screen, lines[1], FONT_MAPEL, label_color,
                               rx + self.w // 2, ry + self.h // 2 + 9)

    def get_rect(self):
        return pygame.Rect(int(self.x), int(self.y), self.w, self.h)

# ─────────────────────────────────────────
#  PLAYER CLASS
# ─────────────────────────────────────────
PLAYER_W, PLAYER_H = 52, 52

class Player:
    def __init__(self):
        self.x = WIDTH // 2 - PLAYER_W // 2
        self.y = HEIGHT - 110
        self.speed = 6
        self.invincible = 0
        self.trail = []

    def update(self, keys):
        if keys[pygame.K_LEFT] or keys[pygame.K_a]:
            self.x -= self.speed
        if keys[pygame.K_RIGHT] or keys[pygame.K_d]:
            self.x += self.speed
        self.x = max(8, min(WIDTH - PLAYER_W - 8, self.x))
        if self.invincible > 0:
            self.invincible -= 1

        cx = self.x + PLAYER_W // 2
        cy = self.y + PLAYER_H // 2
        self.trail.append((cx, cy))
        if len(self.trail) > 12:
            self.trail.pop(0)

    def draw(self, tick):
        # Draw trail
        for i, (tx, ty) in enumerate(self.trail):
            alpha = i / len(self.trail)
            sz = max(2, int(8 * alpha))
            c = lerp_color(C_BG, C_ACCENT, alpha * 0.6)
            pygame.draw.circle(screen, c, (int(tx), int(ty)), sz)

        rx, ry = int(self.x), int(self.y)
        cx = rx + PLAYER_W // 2
        cy = ry + PLAYER_H // 2

        if self.invincible > 0 and (self.invincible // 4) % 2 == 0:
            return  # blink when invincible

        # Glow aura
        pulse = 0.4 + 0.3 * math.sin(tick * 0.12)
        glow_r = int(30 + 10 * pulse)
        glow_surf = pygame.Surface((glow_r * 2, glow_r * 2), pygame.SRCALPHA)
        pygame.draw.circle(glow_surf, (*C_ACCENT, 40), (glow_r, glow_r), glow_r)
        screen.blit(glow_surf, (cx - glow_r, cy - glow_r))

        # Body: rounded square (backpack shape)
        body_rect = pygame.Rect(rx + 8, ry + 16, 36, 32)
        draw_rounded_rect(screen, C_PANEL, body_rect, radius=7)
        draw_rounded_rect(screen, (0, 0, 0, 0), body_rect, radius=7,
                          border=2, border_color=C_ACCENT)

        # Head
        pygame.draw.circle(screen, C_ACCENT2, (cx, ry + 13), 13)
        pygame.draw.circle(screen, C_BORDER, (cx, ry + 13), 13, 2)

        # Eyes
        pygame.draw.circle(screen, C_BG, (cx - 4, ry + 11), 3)
        pygame.draw.circle(screen, C_BG, (cx + 4, ry + 11), 3)
        pygame.draw.circle(screen, C_WHITE, (cx - 4, ry + 11), 2)
        pygame.draw.circle(screen, C_WHITE, (cx + 4, ry + 11), 2)

        # Backpack straps
        pygame.draw.line(screen, C_ACCENT, (rx + 16, ry + 20), (rx + 16, ry + 44), 3)
        pygame.draw.line(screen, C_ACCENT, (rx + 36, ry + 20), (rx + 36, ry + 44), 3)

    def get_rect(self):
        return pygame.Rect(int(self.x) + 10, int(self.y) + 14, 32, 36)

# ─────────────────────────────────────────
#  HUD
# ─────────────────────────────────────────
def draw_hud(score, lives, level, target_idx, tick):
    # Top bar background
    bar_rect = pygame.Rect(0, 0, WIDTH, 58)
    pygame.draw.rect(screen, C_PANEL, bar_rect)
    pygame.draw.line(screen, C_BORDER, (0, 58), (WIDTH, 58), 2)

    # Score
    draw_text(screen, f"SKOR", FONT_TINY, C_DIM, 14, 6)
    draw_text(screen, f"{score}", FONT_BIG, C_GOLD, 14, 20)

    # Level
    draw_text(screen, f"LV", FONT_TINY, C_DIM, 200, 6)
    draw_text(screen, f"{level}", FONT_BIG, C_ACCENT2, 200, 20)

    # Hearts
    for i in range(3):
        hx = WIDTH - 30 - i * 32
        hy = 12
        color = C_HEART if i < lives else C_DIM
        # Simple heart using text symbol
        s = FONT_MED.render("♥", True, color)
        screen.blit(s, (hx, hy))

    # Target banner
    t_name, t_color = MAPEL_LIST[target_idx]
    banner_w = 260
    banner_rect = pygame.Rect(WIDTH // 2 - banner_w // 2, HEIGHT - 74, banner_w, 42)
    pulse = 0.6 + 0.4 * math.sin(tick * 0.08)
    bg_color = lerp_color(C_GOOD_DIM, (20, 70, 40), pulse)
    draw_rounded_rect(screen, bg_color, banner_rect, radius=10)
    draw_rounded_rect(screen, (0, 0, 0, 0), banner_rect, radius=10,
                      border=2, border_color=C_GOOD)

    draw_text_centered(screen, "▶ TANGKAP ◀", FONT_TINY, C_DIM,
                       WIDTH // 2, HEIGHT - 68)
    lines = t_name.split(" ")
    if len(lines) == 1:
        draw_text_centered(screen, t_name, FONT_MED, C_GOOD,
                           WIDTH // 2, HEIGHT - 52)
    else:
        draw_text_centered(screen, lines[0] + " " + lines[1], FONT_MED, C_GOOD,
                           WIDTH // 2, HEIGHT - 52)

    # Bottom bar
    pygame.draw.line(screen, C_BORDER, (0, HEIGHT - 80), (WIDTH, HEIGHT - 80), 1)

    # Credit
    draw_text(screen, "by Abbilefa", FONT_TINY, C_DIM, WIDTH - 82, HEIGHT - 20)

# ─────────────────────────────────────────
#  SCREEN: TITLE
# ─────────────────────────────────────────
def draw_title_screen(tick):
    screen.fill(C_BG)
    draw_stars(tick)

    # Grid lines decoration
    for i in range(0, WIDTH, 40):
        pygame.draw.line(screen, (15, 25, 60), (i, 0), (i, HEIGHT), 1)
    for j in range(0, HEIGHT, 40):
        pygame.draw.line(screen, (15, 25, 60), (0, j), (WIDTH, j), 1)

    # Big glow circle behind title
    glow_surf = pygame.Surface((300, 300), pygame.SRCALPHA)
    pulse = 0.3 + 0.2 * math.sin(tick * 0.05)
    pygame.draw.circle(glow_surf, (*C_BORDER, int(30 * pulse)), (150, 150), 150)
    screen.blit(glow_surf, (WIDTH // 2 - 150, 80))

    # Title
    draw_text_centered(screen, "MAPEL", FONT_TITLE, C_ACCENT2, WIDTH // 2, 170)
    draw_text_centered(screen, "DODGE", FONT_TITLE, C_ACCENT, WIDTH // 2, 220)

    # Subtitle
    pygame.draw.line(screen, C_BORDER, (80, 252), (WIDTH - 80, 252), 2)
    draw_text_centered(screen, "Hindari mapel yang salah!", FONT_SM, C_DIM,
                       WIDTH // 2, 268)
    draw_text_centered(screen, "Tangkap mapel yang ditunjuk!", FONT_SM, C_WHITE,
                       WIDTH // 2, 290)

    # Controls card
    card_rect = pygame.Rect(60, 320, WIDTH - 120, 160)
    draw_rounded_rect(screen, C_PANEL, card_rect, radius=14)
    draw_rounded_rect(screen, (0, 0, 0, 0), card_rect, radius=14,
                      border=2, border_color=C_BORDER)

    draw_text_centered(screen, "KONTROL", FONT_SM, C_ACCENT, WIDTH // 2, 342)
    controls = [
        ("←  /  A", "Gerak Kiri"),
        ("→  /  D", "Gerak Kanan"),
        ("SPACE",   "Pause / Lanjut"),
        ("ENTER",   "Mulai / Restart"),
    ]
    for i, (key, desc) in enumerate(controls):
        y = 362 + i * 26
        draw_text(screen, key,  FONT_SM, C_GOLD,   80,  y)
        draw_text(screen, "—", FONT_SM, C_DIM,    175, y)
        draw_text(screen, desc, FONT_SM, C_WHITE, 195, y)

    # Press enter blink
    if (tick // 30) % 2 == 0:
        draw_text_centered(screen, "[ ENTER untuk Mulai ]", FONT_MED, C_ACCENT,
                           WIDTH // 2, 510)

    # Team credit
    draw_text_centered(screen, "by  Abbilefa", FONT_SM, C_DIM, WIDTH // 2, 545)

    # Floating mapel blocks preview
    for i, offset in enumerate([-160, -60, 40, 140]):
        bx = WIDTH // 2 + offset
        by = 590 + int(10 * math.sin(tick * 0.07 + i))
        m_name, m_color = MAPEL_LIST[i * 3]
        mini_rect = pygame.Rect(bx - 44, by - 18, 88, 36)
        bg = lerp_color(C_PANEL, m_color, 0.2)
        draw_rounded_rect(screen, bg, mini_rect, radius=7)
        draw_rounded_rect(screen, (0, 0, 0, 0), mini_rect, radius=7,
                          border=2, border_color=m_color)
        lines = m_name.split(" ")
        if len(lines) == 1:
            draw_text_centered(screen, m_name, FONT_TINY, m_color, bx, by)
        else:
            draw_text_centered(screen, lines[0], FONT_TINY, m_color, bx, by - 8)
            draw_text_centered(screen, lines[1], FONT_TINY, m_color, bx, by + 8)

# ─────────────────────────────────────────
#  SCREEN: GAME OVER
# ─────────────────────────────────────────
def draw_gameover_screen(score, highscore, tick):
    # Darken overlay
    overlay = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
    overlay.fill((0, 0, 0, 180))
    screen.blit(overlay, (0, 0))

    card_rect = pygame.Rect(60, 160, WIDTH - 120, 340)
    draw_rounded_rect(screen, C_PANEL, card_rect, radius=18)
    draw_rounded_rect(screen, (0, 0, 0, 0), card_rect, radius=18,
                      border=3, border_color=C_DANGER)

    draw_text_centered(screen, "GAME OVER", FONT_TITLE, C_DANGER, WIDTH // 2, 220)
    pygame.draw.line(screen, C_DANGER, (100, 258), (WIDTH - 100, 258), 2)

    draw_text_centered(screen, "SKOR KAMU", FONT_SM, C_DIM, WIDTH // 2, 282)
    draw_text_centered(screen, str(score), FONT_TITLE, C_GOLD, WIDTH // 2, 318)

    draw_text_centered(screen, f"REKOR TERTINGGI  :  {highscore}", FONT_SM, C_ACCENT,
                       WIDTH // 2, 362)

    if score >= highscore and score > 0:
        pulse = 0.5 + 0.5 * math.sin(tick * 0.15)
        c = lerp_color(C_GOLD, C_WHITE, pulse)
        draw_text_centered(screen, "★ REKOR BARU! ★", FONT_MED, c, WIDTH // 2, 390)

    if (tick // 30) % 2 == 0:
        draw_text_centered(screen, "[ ENTER untuk Main Lagi ]", FONT_SM, C_ACCENT,
                           WIDTH // 2, 440)

    draw_text_centered(screen, "by Abbilefa", FONT_TINY, C_DIM, WIDTH // 2, 480)

# ─────────────────────────────────────────
#  SCREEN: PAUSE
# ─────────────────────────────────────────
def draw_pause_screen():
    overlay = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
    overlay.fill((0, 0, 0, 150))
    screen.blit(overlay, (0, 0))

    card_rect = pygame.Rect(100, 240, WIDTH - 200, 180)
    draw_rounded_rect(screen, C_PANEL, card_rect, radius=14)
    draw_rounded_rect(screen, (0, 0, 0, 0), card_rect, radius=14,
                      border=2, border_color=C_ACCENT)

    draw_text_centered(screen, "⏸  PAUSE", FONT_BIG, C_ACCENT2, WIDTH // 2, 300)
    draw_text_centered(screen, "Tekan SPACE untuk Lanjut", FONT_SM, C_DIM,
                       WIDTH // 2, 360)

# ─────────────────────────────────────────
#  FLOATING SCORE TEXT
# ─────────────────────────────────────────
float_texts = []

def spawn_float_text(x, y, text, color):
    float_texts.append({"x": x, "y": y, "text": text, "color": color,
                         "life": 1.0, "vy": -2.5})

def update_draw_float_texts():
    dead = []
    for ft in float_texts:
        ft["y"] += ft["vy"]
        ft["life"] -= 0.025
        if ft["life"] <= 0:
            dead.append(ft)
            continue
        alpha = ft["life"]
        c = tuple(int(ft["color"][i] * min(1, alpha * 2)) for i in range(3))
        s = FONT_MED.render(ft["text"], True, c)
        screen.blit(s, (int(ft["x"] - s.get_width() // 2), int(ft["y"])))
    for ft in dead:
        float_texts.remove(ft)

# ─────────────────────────────────────────
#  MAIN GAME
# ─────────────────────────────────────────
def main():
    state = "title"
    score = 0
    highscore = 0
    lives = 3
    level = 1
    tick = 0
    paused = False

    player = Player()
    blocks = []
    target_idx = random.randint(0, len(MAPEL_LIST) - 1)

    spawn_timer = 0
    base_speed = 2.8
    block_speed = base_speed
    spawn_interval = 90  # frames between spawns
    blocks_per_spawn = 1

    def reset_game():
        nonlocal score, lives, level, blocks, target_idx
        nonlocal spawn_timer, block_speed, spawn_interval, blocks_per_spawn
        score = 0
        lives = 3
        level = 1
        blocks = []
        particles.clear()
        float_texts.clear()
        target_idx = random.randint(0, len(MAPEL_LIST) - 1)
        block_speed = base_speed
        spawn_interval = 90
        blocks_per_spawn = 1
        spawn_timer = 0
        player.x = WIDTH // 2 - PLAYER_W // 2
        player.invincible = 0
        player.trail.clear()

    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

            if event.type == pygame.KEYDOWN:
                if state == "title":
                    if event.key == pygame.K_RETURN:
                        reset_game()
                        state = "game"

                elif state == "game":
                    if event.key == pygame.K_SPACE:
                        paused = not paused
                    if event.key == pygame.K_RETURN and paused:
                        paused = False

                elif state == "gameover":
                    if event.key == pygame.K_RETURN:
                        reset_game()
                        state = "game"

        # ── UPDATE ──────────────────────────────
        tick += 1

        if state == "game" and not paused:
            keys = pygame.key.get_pressed()
            player.update(keys)

            # Spawn blocks
            spawn_timer += 1
            if spawn_timer >= spawn_interval:
                spawn_timer = 0
                used_x = []
                for _ in range(blocks_per_spawn):
                    # Ensure at least 1 target per wave randomly
                    is_target_block = (random.random() < 0.35)
                    midx = random.randint(0, len(MAPEL_LIST) - 1)
                    while is_target_block and midx == target_idx:
                        midx = random.randint(0, len(MAPEL_LIST) - 1)
                    if is_target_block:
                        midx = target_idx

                    b = Block(block_speed, is_target_block, midx)
                    # Avoid x overlap
                    attempts = 0
                    while any(abs(b.x - ux) < BLOCK_W + 10 for ux in used_x) and attempts < 10:
                        b.x = random.randint(20, WIDTH - 20 - BLOCK_W)
                        attempts += 1
                    used_x.append(b.x)
                    blocks.append(b)

            # Update blocks & check collision
            p_rect = player.get_rect()
            for b in blocks:
                b.update(tick)
                if not b.alive:
                    continue

                if b.get_rect().colliderect(p_rect):
                    if b.is_target:
                        score += 10
                        b.alive = False
                        cx = b.x + b.w // 2
                        cy = b.y + b.h // 2
                        spawn_particles(cx, cy, C_GOOD, 15)
                        spawn_float_text(cx, cy - 20, "+10", C_GOOD)
                        # Change target
                        old = target_idx
                        while target_idx == old:
                            target_idx = random.randint(0, len(MAPEL_LIST) - 1)
                    else:
                        if player.invincible == 0:
                            lives -= 1
                            b.alive = False
                            player.invincible = 90
                            cx = b.x + b.w // 2
                            cy = b.y + b.h // 2
                            spawn_particles(cx, cy, C_DANGER, 10)
                            spawn_float_text(cx, cy - 20, "-NYAWA", C_DANGER)
                            if lives <= 0:
                                highscore = max(highscore, score)
                                state = "gameover"

            blocks = [b for b in blocks if b.alive]

            # Level scaling every 50 pts
            new_level = 1 + score // 50
            if new_level != level:
                level = new_level
                block_speed = base_speed + (level - 1) * 0.4
                spawn_interval = max(35, 90 - (level - 1) * 8)
                blocks_per_spawn = min(4, 1 + (level - 1) // 2)

        # ── DRAW ────────────────────────────────
        screen.fill(C_BG)
        draw_stars(tick)

        # subtle grid
        for i in range(0, WIDTH, 40):
            pygame.draw.line(screen, (12, 20, 48), (i, 0), (i, HEIGHT), 1)
        for j in range(0, HEIGHT, 40):
            pygame.draw.line(screen, (12, 20, 48), (0, j), (WIDTH, j), 1)

        if state == "title":
            draw_title_screen(tick)

        elif state == "game":
            for b in blocks:
                b.draw(tick)
            update_draw_particles()
            player.draw(tick)
            update_draw_float_texts()
            draw_hud(score, lives, level, target_idx, tick)
            if paused:
                draw_pause_screen()

        elif state == "gameover":
            for b in blocks:
                b.draw(tick)
            player.draw(tick)
            draw_hud(score, lives, level, target_idx, tick)
            draw_gameover_screen(score, highscore, tick)

        pygame.display.flip()
        clock.tick(FPS)

if __name__ == "__main__":
    main()
