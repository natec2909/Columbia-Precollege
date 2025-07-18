import pygame
import sys

# Initialize
pygame.init()
WIDTH, HEIGHT = 1280, 720
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Fighting Game")
clock = pygame.time.Clock()
FPS = 60

# Load and scale background image
background = pygame.image.load("background_image.png").convert()
background = pygame.transform.scale(background, (WIDTH, HEIGHT))

# Load idle sprites and scale them
player1_sprite = pygame.image.load("necromancer_3_idle_transparent.png").convert_alpha()
player1_sprite = pygame.transform.scale(player1_sprite, (150, 150))

player2_sprite = pygame.image.load("necromancer_4_idle_transparent.png").convert_alpha()
player2_sprite = pygame.transform.scale(player2_sprite, (150, 150))

player3_sprite = pygame.image.load("necromancer_5_idle_transparent.png").convert_alpha()
player3_sprite = pygame.transform.scale(player3_sprite, (150, 150))

# Colors
RED = (255, 0, 0)
YELLOW = (255, 255, 0)
GREEN = (0, 255, 0)
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)

# Game settings
GRAVITY = 1
JUMP_POWER = -20
PLAYER_SPEED = 8
MAX_HEALTH = 100
MAX_LIVES = 3
ATTACK_COOLDOWN = 250  # milliseconds

def draw_text(text, x, y, size=36, color=(255, 255, 255)):
    font = pygame.font.SysFont("Arial", size)
    surface = font.render(text, True, color)
    screen.blit(surface, (x, y))

def get_sprite_for_color(color):
    if color == RED:
        return player1_sprite
    elif color == YELLOW:
        return player2_sprite
    elif color == GREEN:
        return player3_sprite
    return None

def get_character_name(color):
    if color == RED:
        return "Red Devil"
    elif color == YELLOW:
        return "Icy Queen"
    elif color == GREEN:
        return "Sharp Shooter"
    return "Unknown"

def load_attack_images(folder, base_name, count=12):
    images = []
    for i in range(1, count + 1):
        path = f"{folder}/{base_name}_attack_{i}.png"
        img = pygame.image.load(path).convert_alpha()
        img = pygame.transform.scale(img, (150, 150))
        images.append(img)
    return images

class Player:
    def __init__(self, x, y, color, controls, platforms, sprite_image, sprite_offset_y=10):
        self.rect = pygame.Rect(x, y, 50, 50)
        self.color = color
        self.controls = controls
        self.platforms = platforms
        self.vel_y = 0
        self.on_ground = False
        self.facing_right = True
        self.health = MAX_HEALTH
        self.lives = MAX_LIVES
        self.attack_rect = None
        self.attack_timer = 0
        self.invincible_timer = 0
        self.last_hit_time = 0
        self.heal_tick_timer = 0
        self.sprite_image = sprite_image
        self.sprite_offset_y = sprite_offset_y
        self.attack_cooldown_timer = 0

        # Dash related
        self.dash_cooldown_timer = 0
        self.dash_active = False
        self.dash_distance = 150  # total dash pixels
        self.dash_duration = 200  # dash duration in milliseconds
        self.dash_start_time = 0
        self.dash_direction = 1
        self.dash_speed = self.dash_distance / (self.dash_duration / (1000 / FPS))  # pixels per frame

        # Load attack animations
        if color == RED:
            self.attack_images = load_attack_images("Slashing 3", "necromancer_3", 12)
        elif color == YELLOW:
            self.attack_images = load_attack_images("Slashing 4", "necromancer_4", 12)
        elif color == GREEN:
            self.attack_images = load_attack_images("Slashing 5", "necromancer_5", 12)
        else:
            self.attack_images = []

        self.is_attacking = False
        self.current_attack_frame = 0
        self.attack_frame_timer = 0
        self.ATTACK_FRAME_DURATION = 30  # milliseconds per frame

    def move(self, keys):
        if self.dash_active:
            # During dash: move smoothly in dash direction
            self.rect.x += self.dash_speed * self.dash_direction
            # End dash after duration
            if pygame.time.get_ticks() - self.dash_start_time >= self.dash_duration:
                self.dash_active = False
        else:
            dx = 0
            if keys[self.controls['left']]:
                dx -= PLAYER_SPEED
                self.facing_right = False
            if keys[self.controls['right']]:
                dx += PLAYER_SPEED
                self.facing_right = True
            if keys[self.controls['jump']] and self.on_ground:
                self.vel_y = JUMP_POWER
            self.rect.x += dx

        # Keep player inside screen bounds
        if self.rect.left < 0:
            self.rect.left = 0
        if self.rect.right > WIDTH:
            self.rect.right = WIDTH

    def start_dash(self):
        current_time = pygame.time.get_ticks()
        if current_time - self.dash_cooldown_timer >= 3500 and not self.dash_active:
            self.dash_active = True
            self.dash_start_time = current_time
            self.dash_cooldown_timer = current_time
            self.dash_direction = 1 if self.facing_right else -1
            self.dash_distance = 300  # <-- Increased Dash Distance
            self.dash_speed = self.dash_distance / (self.dash_duration / (1000 / FPS))

    def apply_gravity(self):
        self.vel_y += GRAVITY
        self.rect.y += self.vel_y

    def check_collision(self):
        self.on_ground = False
        for platform in self.platforms:
            if self.rect.colliderect(platform) and self.vel_y >= 0:
                self.rect.bottom = platform.top
                self.vel_y = 0
                self.on_ground = True

    def attack(self):
        current_time = pygame.time.get_ticks()
        if current_time - self.attack_cooldown_timer >= ATTACK_COOLDOWN and not self.is_attacking:
            self.is_attacking = True
            self.current_attack_frame = 0
            self.attack_frame_timer = current_time
            self.attack_cooldown_timer = current_time

            # Invisible hitbox created at attack start
            offset = 20
            width = 60
            height = 70
            if self.facing_right:
                attack_x = self.rect.right + offset
            else:
                attack_x = self.rect.left - width - offset
            self.attack_rect = pygame.Rect(attack_x, self.rect.centery - height // 2, width, height)
            self.attack_timer = 10  # frames hitbox stays active

    def take_damage(self, amount):
        if self.invincible_timer == 0:
            self.health -= amount
            self.last_hit_time = pygame.time.get_ticks()
            self.invincible_timer = 30
            if self.health <= 0:
                self.lives -= 1
                self.health = MAX_HEALTH
                self.rect.topleft = (100, 100)

    def heal(self):
        current_time = pygame.time.get_ticks()
        time_since_hit = current_time - self.last_hit_time
        if time_since_hit > 3000 and self.health < MAX_HEALTH:
            self.heal_tick_timer += 1
            if self.heal_tick_timer >= 120:
                self.health += 5
                if self.health > MAX_HEALTH:
                    self.health = MAX_HEALTH
                self.heal_tick_timer = 0
        else:
            self.heal_tick_timer = 0

    def update(self, keys, other_player):
        self.move(keys)
        self.apply_gravity()
        self.check_collision()
        self.heal()

        if self.invincible_timer > 0:
            self.invincible_timer -= 1

        # Animate attack frames
        if self.is_attacking:
            now = pygame.time.get_ticks()
            if now - self.attack_frame_timer > self.ATTACK_FRAME_DURATION:
                self.attack_frame_timer = now
                self.current_attack_frame += 1

                if self.current_attack_frame >= len(self.attack_images):
                    self.is_attacking = False
                    self.current_attack_frame = 0
                    self.attack_rect = None

        # Attack hitbox timer countdown
        if self.attack_rect and self.attack_timer > 0:
            self.attack_timer -= 1
            if self.attack_timer == 0:
                self.attack_rect = None

        # Deal damage if hitbox overlaps other player
        if self.attack_rect and self.attack_rect.colliderect(other_player.rect):
            other_player.take_damage(10)

    def draw(self):
        if self.sprite_image:
            if self.is_attacking and self.attack_images:
                frame = self.attack_images[self.current_attack_frame]
            else:
                frame = self.sprite_image

            flipped_image = pygame.transform.flip(frame, not self.facing_right, False)
            image_rect = frame.get_rect()
            image_x = self.rect.centerx - image_rect.width // 2
            image_y = self.rect.bottom - image_rect.height + 25
            screen.blit(flipped_image, (image_x, image_y))
        else:
            pygame.draw.rect(screen, self.color, self.rect)

        # Attack hitbox is invisible (no rect drawn)

def draw_health_bar(player, x, y):
    bar_width = 200
    bar_height = 20
    segment_width = bar_width // 10
    pygame.draw.rect(screen, GREEN, (x, y, bar_width, bar_height))
    pygame.draw.rect(screen, WHITE, (x - 2, y - 2, bar_width + 4, bar_height + 4), 2)
    segments_to_cover = (MAX_HEALTH - player.health) // 10
    for i in range(segments_to_cover):
        segment_x = x + bar_width - (i + 1) * segment_width
        pygame.draw.rect(screen, RED, (segment_x, y, segment_width, bar_height))

def character_selection():
    selecting_p1 = True
    selecting_p2 = False
    player1_color = None
    player2_color = None

    color_map = {
        pygame.K_1: RED,
        pygame.K_2: YELLOW,
        pygame.K_3: GREEN
    }

    while True:
        screen.fill((0, 0, 50))

        if selecting_p1:
            draw_text("Player 1: Press 1 for Red Devil, 2 for Icy Queen, or 3 for Sharp Shooter", 60, 50)
        elif selecting_p2:
            draw_text("Player 2: Press 1 for Red Devil, 2 for Icy Queen, or 3 for Sharp Shooter", 60, 50)
        else:
            return player1_color, player2_color

        red_devil_pos = (150, 200)
        icy_queen_pos = (540, 200)
        sharp_shooter_pos = (930, 200)

        screen.blit(player1_sprite, red_devil_pos)
        screen.blit(player2_sprite, icy_queen_pos)
        screen.blit(player3_sprite, sharp_shooter_pos)

        draw_text("Red Devil", red_devil_pos[0] + player1_sprite.get_width() // 2 - 40,
                  red_devil_pos[1] + player1_sprite.get_height() + 10)
        draw_text("Icy Queen", icy_queen_pos[0] + player2_sprite.get_width() // 2 - 40,
                  icy_queen_pos[1] + player2_sprite.get_height() + 10)
        draw_text("Sharp Shooter", sharp_shooter_pos[0] + player3_sprite.get_width() // 2 - 60,
                  sharp_shooter_pos[1] + player3_sprite.get_height() + 10)

        pygame.display.flip()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if selecting_p1 and event.key in color_map:
                    player1_color = color_map[event.key]
                    selecting_p1 = False
                    selecting_p2 = True
                elif selecting_p2 and event.key in color_map:
                    player2_color = color_map[event.key]
                    selecting_p2 = False

def main():
    player1_color, player2_color = character_selection()

    platforms = [
        pygame.Rect(0, HEIGHT - 40, WIDTH, 40),
        pygame.Rect(150, 500, 300, 20),
        pygame.Rect(480, 380, 300, 20),
        pygame.Rect(830, 500, 300, 20)
    ]

    player1_controls = {
        'left': pygame.K_a,
        'right': pygame.K_d,
        'jump': pygame.K_w,
        'dash': pygame.K_s,
        'attack': pygame.K_e
    }
    player2_controls = {
        'left': pygame.K_LEFT,
        'right': pygame.K_RIGHT,
        'jump': pygame.K_UP,
        'dash': pygame.K_DOWN,
        'attack': pygame.K_SPACE
    }

    player1 = Player(100, HEIGHT - 90, player1_color, player1_controls, platforms, get_sprite_for_color(player1_color))
    player2 = Player(1100, HEIGHT - 90, player2_color, player2_controls, platforms, get_sprite_for_color(player2_color))

    game_over = False

    while True:
        screen.blit(background, (0, 0))
        keys = pygame.key.get_pressed()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if not game_over and event.type == pygame.KEYDOWN:
                if event.key == player1.controls['attack']:
                    player1.attack()
                if event.key == player2.controls['attack']:
                    player2.attack()
                if event.key == player1.controls['dash']:
                    player1.start_dash()
                if event.key == player2.controls['dash']:
                    player2.start_dash()
            elif game_over and event.type == pygame.KEYDOWN:
                if event.key == pygame.K_RETURN:
                    main()
                elif event.key == pygame.K_q:
                    pygame.quit()
                    sys.exit()

        if not game_over:
            player1.update(keys, player2)
            player2.update(keys, player1)

        for platform in platforms:
            pygame.draw.rect(screen, BLACK, platform)

        player1.draw()
        player2.draw()

        draw_text(get_character_name(player1_color), 20, 10)
        draw_text(get_character_name(player2_color), WIDTH - 240, 10)

        draw_health_bar(player1, 20, 50)
        draw_health_bar(player2, WIDTH - 220, 50)

        draw_text(f"Lives: {player1.lives}", 20, 80)
        draw_text(f"Lives: {player2.lives}", WIDTH - 180, 80)

        if player1.lives == 0 or player2.lives == 0:
            game_over = True
            winner = "Player 1" if player2.lives == 0 else "Player 2"
            draw_text(f"{winner} Wins!", WIDTH // 2 - 100, HEIGHT // 2, 48)
            draw_text("Press ENTER to Restart or Q to Quit", WIDTH // 2 - 220, HEIGHT // 2 + 60)

        pygame.display.flip()
        clock.tick(FPS)

if __name__ == "__main__":
    main()