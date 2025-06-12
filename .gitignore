import random
page = "Welcome"
keys_pressed = set()
cols, rows = 40, 40
cell_size = 15
grid = []
collectibles = []
num_collected = 0
total_collectibles = 1
collected_since_spawn = 0
max_collectibles_on_map = 10
respawn_threshold = 3
respawn_amount = 3
flashing_collectible_index = 0
difficulty_increased = 1
player_x = player_y = 0
player_size = cell_size//2
player_speed = 2
num_enemies = 30
enemy_positions = []
enemy_dirs = []
enemy_size = cell_size//2
enemy_speed = 1.5
tutorial_steps = [
    {"text": "Press W to move UP", "key": "w"},
    {"text": "Press S to move DOWN", "key": "s"},
    {"text": "Press A to move LEFT", "key": "a"},
    {"text": "Press D to move RIGHT", "key": "d"},
    {"text": "Avoid the enemy by moving!", "key": None}
]
current_tutorial_step = 0
tutorial_completed = False
timer = 0
time_limit = 120
def setup():
    global grid, player_x, player_y, num_collected, flashing_collectible_index, timer
    size(800, 700)
    frameRate(30)
    noStroke()
    grid = generate_maze(cols, rows)
    player_x, player_y = cell_center(cols // 2, rows // 2)
    place_enemies(num_enemies)
    place_collectibles(total_collectibles)
    num_collected = 0
    timer = 0
    if collectibles:
        flashing_collectible_index = random.randint(0, len(collectibles) - 1)
        
def generate_maze(w, h):
    maze = [[1 for _ in range(h)] for _ in range(w)]
    visited = [[False for _ in range(h)] for _ in range(w)]

    def carve(x, y):
        maze[x][y] = 0
        visited[x][y] = True
        directions = [(2, 0), (-2, 0), (0, 2), (0, -2)]
        random.shuffle(directions)
        for dx, dy in directions:
            nx, ny = x + dx, y + dy
            if 1 <= nx < w - 1 and 1 <= ny < h - 1 and not visited[nx][ny]:
                maze[x + dx // 2][y + dy // 2] = 0
                carve(nx, ny)

    carve(1, 1)
    for i in range(w):
        maze[i][0] = maze[i][h - 1] = 1
    for j in range(h):
        maze[0][j] = maze[w - 1][j] = 1
    maze[cols // 2][rows // 2] = 0
    wall_cells = [(x, y) for x in range(1, w - 1) for y in range(1, h - 1) if maze[x][y] == 1]
    for x, y in random.sample(wall_cells, len(wall_cells) // 5):
        maze[x][y] = 0
    return maze

def cell_center(cx, cy):
    return cx * cell_size + cell_size // 2, cy * cell_size + cell_size // 2

def place_enemies(n):
    global enemy_positions, enemy_dirs
    enemy_positions = []
    enemy_dirs = []
    placed = 0
    while placed < n:
        ex = random.randint(1, cols - 2)
        ey = random.randint(1, rows - 2)
        if (ex != cols // 2 or ey != rows // 2) and grid[ex][ey] == 0:
            pos = cell_center(ex, ey)
            if all(dist(pos[0], pos[1], e[0], e[1]) > 20 for e in enemy_positions):
                enemy_positions.append(pos)
                enemy_dirs.append(random.choice(['up', 'down', 'left', 'right']))
                placed += 1

def place_collectibles(n):
    global collectibles
    collectibles = []
    while len(collectibles) < n:
        cx = random.randint(1, cols - 2)
        cy = random.randint(1, rows - 2)
        if grid[cx][cy] == 0:
            px, py = cell_center(cx, cy)
            if abs(px - player_x) > 20 or abs(py - player_y) > 20:
                collectibles.append((px, py))

def can_move(x, y, size):
    for dx in [-size // 2, size // 2]:
        for dy in [-size // 2, size // 2]:
            gx = int(x + dx) // cell_size
            gy = int(y + dy) // cell_size
            if gx < 0 or gx >= cols or gy < 0 or gy >= rows or grid[gx][gy] == 1:
                return False
    return True

def draw():
    global page, current_tutorial_step, tutorial_completed, timer

    background(102)
    if page == "Welcome":
        draw_menu()
    elif page == "Page1" or page == "InteractiveTutorial":

        if frameCount % 30 == 0:
            timer += 1
        
        time_remaining = max(0, time_limit - timer)
        fill(255)
        textSize(20)
        textAlign(LEFT, BOTTOM)
        text("Time Left: " + str(time_remaining), 10, height - 10)

        move_player()
        if page == "Page1":
            move_enemies()
        offset_x = (width - cols * cell_size) // 2
        offset_y = (height - rows * cell_size) // 2 - 30
        draw_grid(offset_x, offset_y)

        if collectibles:
            cx, cy = collectibles[flashing_collectible_index]
            if (frameCount // 15) % 2 == 0:
                fill(0, 255, 0)
            else:
                fill(0)
            rect(offset_x + cx - 5, offset_y + cy - 5, 10, 10)

        fill(255, 0, 0)
        rectMode(CENTER)
        rect(offset_x + player_x, offset_y + player_y, player_size, player_size)

        fill(0, 0, 255)
        for ex, ey in enemy_positions:
            rect(offset_x + ex, offset_y + ey, enemy_size, enemy_size)
        rectMode(CORNER)

        fill(255)
        textAlign(LEFT, TOP)
        textSize(18)
        text("Collected: " + str(num_collected) + "/" + str(total_collectibles), 10, 10)

        if time_remaining <= 0:
            page = "Give Up"

        for ex, ey in enemy_positions:
            if is_collision(player_x, player_y, player_size, ex, ey, enemy_size):
                page = "Give Up"

        if num_collected >= total_collectibles and difficulty_increased != 10:
            increase_difficulty()
            difficulty_increased +=1
        
        if num_collected == total_collectibles and difficulty_increased == 10:
            page = "Win"

        if page == "Page1":
            draw_button((width - 200) // 2, height - 80, 200, 50, "Give Up", False)

        if page == "InteractiveTutorial":
            fill(0)
            textAlign(CENTER, CENTER)
            textSize(30)
            if current_tutorial_step < len(tutorial_steps):
                text(tutorial_steps[current_tutorial_step]["text"], width // 2, height-35)
            else:
                tutorial_completed = True
                page = "tutorial_complete"
            

    elif page == "Give Up":
        background('#FF0000')
        textAlign(CENTER, CENTER)
        fill('#080808')
        textSize(120)
        text("GAME OVER", width // 2, height // 2 - 50)
        textSize(30)
        text("You were caught by the enemy or ran out of time!", width // 2, height // 2 + 20)
    elif page == "tutorial_complete":
        background('#C5D0FA')
        textAlign(CENTER, CENTER)
        fill('#080808')
        textSize(60)
        text("Tutorial Complete!", width // 2, height // 2 - 50)
        textSize(30)
        text("Click for the real game!", width // 2, height // 2 + 20)


    elif page == "Tutorial":
        background(220)
        fill(0)
        textAlign(CENTER)
        textSize(40)
        text("Tutorial", width // 2, 80)
        textSize(25)
        lines = [
            "Controls:",
            "- Use W, A, S, D to move.",
            "- Avoid walls and the enemy.",
            "- Only collect the flashing green cube!",
            "- If the enemy touches you, you lose.",
            "",
            "Click to play!"
        ]
        for i, line in enumerate(lines):
            text(line, width // 2, 150 + i * 35)

    elif page == "Win":
        background(0, 200, 0)
        fill(255)
        textAlign(CENTER, CENTER)
        textSize(60)
        text("You collected all cubes!", width // 2, height // 2)
        textSize(30)
        text("Click to return to menu.", width // 2, height // 2 + 60)

def draw_grid(offset_x, offset_y):
    for i in range(cols):
        for j in range(rows):
            fill(200 if grid[i][j] == 0 else 50)
            rect(offset_x + i * cell_size, offset_y + j * cell_size, cell_size, cell_size)

def draw_menu():
    draw_button(50, 250, 200, 50, "Start", 50 < mouseX < 250 and 250 < mouseY < 300)
    draw_button(50, 400, 200, 50, "Quit", 50 < mouseX < 250 and 400 < mouseY < 450)
    draw_button(50, 500, 200, 50, "Tutorial", 50 < mouseX < 250 and 500 < mouseY < 550)
    draw_button(50, 600, 200, 50, "Interactive Tutorial", 50 < mouseX < 250 and 600 < mouseY < 650)

def draw_button(x, y, w, h, label, hover):
    fill(0, 100, 255) if hover else fill(0, 0, 255)
    rect(x, y, w, h)
    fill(255)
    textSize(30)
    textAlign(CENTER, CENTER)
    text(label, x + w // 2, y + h // 2)

def move_player():
    global player_x, player_y
    dx = dy = 0
    if 'w' in keys_pressed:
        dy -= player_speed
    if 's' in keys_pressed:
        dy += player_speed
    if 'a' in keys_pressed:
        dx -= player_speed
    if 'd' in keys_pressed:
        dx += player_speed
    if can_move(player_x + dx, player_y, player_size):
        player_x += dx
    if can_move(player_x, player_y + dy, player_size):
        player_y += dy
    check_collectibles()

def move_enemies():
    global enemy_positions, enemy_dirs
    for i in range(len(enemy_positions)):
        if frameCount % 20 == 0:
            enemy_dirs[i] = random.choice(['up', 'down', 'left', 'right'])
        dx = dy = 0
        if enemy_dirs[i] == 'up':
            dy = -enemy_speed
        elif enemy_dirs[i] == 'down':
            dy = enemy_speed
        elif enemy_dirs[i] == 'left':
            dx = -enemy_speed
        elif enemy_dirs[i] == 'right':
            dx = enemy_speed
        x, y = enemy_positions[i]
        if can_move(x + dx, y + dy, enemy_size):
            enemy_positions[i] = (x + dx, y + dy)

def is_collision(x1, y1, size1, x2, y2, size2):
    return abs(x1 - x2) < (size1 + size2) / 2 and abs(y1 - y2) < (size1 + size2) / 2

def check_collectibles():
    global num_collected, collectibles, collected_since_spawn, flashing_collectible_index
    if flashing_collectible_index < len(collectibles):
        fx, fy = collectibles[flashing_collectible_index]
        if dist(player_x, player_y, fx, fy) < 10:
            num_collected += 1
            collected_since_spawn += 1
            del collectibles[flashing_collectible_index]
            if collectibles:
                flashing_collectible_index = random.randint(0, len(collectibles) - 1)
            else:
                flashing_collectible_index = 0
    if collected_since_spawn >= respawn_threshold:
        spawn_new_collectibles(min(respawn_amount, max_collectibles_on_map - len(collectibles)))
        collected_since_spawn = 0

def spawn_new_collectibles(n):
    global collectibles, flashing_collectible_index
    spawned = 0
    attempts = 0
    while spawned < n and attempts < 100:
        cx = random.randint(1, cols - 2)
        cy = random.randint(1, rows - 2)
        px, py = cell_center(cx, cy)
        if grid[cx][cy] == 0:
            if dist(px, py, player_x, player_y) > 20 and all(dist(px, py, c[0], c[1]) > 10 for c in collectibles):
                collectibles.append((px, py))
                spawned += 1
        attempts += 1
    if collectibles:
        flashing_collectible_index = random.randint(0, len(collectibles) - 1)

def keyPressed():
    keys_pressed.add(key.lower())
    global current_tutorial_step, page
    if page == "InteractiveTutorial" and current_tutorial_step < len(tutorial_steps):
        step = tutorial_steps[current_tutorial_step]
        if step["key"] is None or key.lower() == step["key"]:
            current_tutorial_step += 1

def keyReleased():
    keys_pressed.discard(key.lower())

def mouseClicked():
    global page, current_tutorial_step, tutorial_completed
    if page == "Welcome":
        if 50 < mouseX < 250:
            if 250 < mouseY < 300:
                setup()
                page = "Page1"
            elif 400 < mouseY < 450:
                exit()
            elif 500 < mouseY < 550:
                page = "Tutorial"
            elif 600 < mouseY < 650:
                setup()
                current_tutorial_step = 0
                tutorial_completed = False
                page = "InteractiveTutorial"
    elif page in ["Give Up", "Tutorial", "Win"]:
        page = "Welcome"
    elif page == "Give Up":
        if width // 2 - 100 < mouseX < width // 2 + 100 and height // 2 + 80 < mouseY < height // 2 + 130:
            setup()
            page = "Page1"
            textSize(30)
            if current_tutorial_step < len(tutorial_steps):
                text(tutorial_steps[current_tutorial_step]["text"], width // 2, height-35)
            else:
                text("Tutorial Complete! Click to return.", width // 2, height-35)
                tutorial_completed = True
          

    elif page == "Give Up":
        background('#FF0000')
        textAlign(CENTER, CENTER)
        fill('#080808')
        textSize(120)
        text("GAME OVER", width // 2, height // 2 - 50)
        textSize(30)
        text("You were caught by the enemy!", width // 2, height // 2 + 20)

    elif page == "Tutorial":
        background(220)
        fill(0)
        textAlign(CENTER)
        textSize(40)
        text("Tutorial", width // 2, 80)
        textSize(25)
        lines = [
            "Controls:",
            "- Use W, A, S, D to move.",
            "- Avoid walls and the enemy.",
            "- Only collect the flashing green cube!",
            "- If the enemy touches you, you lose.",
            "",
            "Click to play!"
        ]
        for i, line in enumerate(lines):
            text(line, width // 2, 150 + i * 35)

    elif page == "Win":
        background(0, 200, 0)
        fill(255)
        textAlign(CENTER, CENTER)
        textSize(60)
        text("You collected all cubes!", width // 2, height // 2)
        textSize(30)
        text("Click to return to menu.", width // 2, height // 2 + 60)
    if time_remaining <= 0:
        page = "Give Up"

def draw_grid(offset_x, offset_y):
    for i in range(cols):
        for j in range(rows):
            fill(200 if grid[i][j] == 0 else 50)
            rect(offset_x + i * cell_size, offset_y + j * cell_size, cell_size, cell_size)

def draw_menu():
    draw_button(50, 250, 200, 50, "Start", 50 < mouseX < 250 and 250 < mouseY < 300)
    draw_button(50, 400, 200, 50, "Quit", 50 < mouseX < 250 and 400 < mouseY < 450)
    draw_button(50, 500, 200, 50, "Tutorial", 50 < mouseX < 250 and 500 < mouseY < 550)
    draw_button(50, 600, 200, 50, "Interactive Tutorial", 50 < mouseX < 250 and 600 < mouseY < 650)

def draw_button(x, y, w, h, label, hover):
    fill(0, 100, 255) if hover else fill(0, 0, 255)
    rect(x, y, w, h)
    fill(255)
    textSize(30)
    textAlign(CENTER, CENTER)
    text(label, x + w // 2, y + h // 2)

def move_player():
    global player_x, player_y
    dx = dy = 0
    if 'w' in keys_pressed:
        dy -= player_speed
    if 's' in keys_pressed:
        dy += player_speed
    if 'a' in keys_pressed:
        dx -= player_speed
    if 'd' in keys_pressed:
        dx += player_speed
    if can_move(player_x + dx, player_y, player_size):
        player_x += dx
    if can_move(player_x, player_y + dy, player_size):
        player_y += dy
    check_collectibles()

def move_enemies():
    global enemy_positions, enemy_dirs
    for i in range(len(enemy_positions)):
        if frameCount % 20 == 0:
            enemy_dirs[i] = random.choice(['up', 'down', 'left', 'right'])
        dx = dy = 0
        if enemy_dirs[i] == 'up':
            dy = -enemy_speed
        elif enemy_dirs[i] == 'down':
            dy = enemy_speed
        elif enemy_dirs[i] == 'left':
            dx = -enemy_speed
        elif enemy_dirs[i] == 'right':
            dx = enemy_speed
        x, y = enemy_positions[i]
        if can_move(x + dx, y + dy, enemy_size):
            enemy_positions[i] = (x + dx, y + dy)

def is_collision(x1, y1, size1, x2, y2, size2):
    return abs(x1 - x2) < (size1 + size2) / 2 and abs(y1 - y2) < (size1 + size2) / 2

def check_collectibles():
    global num_collected, collectibles, collected_since_spawn, flashing_collectible_index
    if flashing_collectible_index < len(collectibles):
        fx, fy = collectibles[flashing_collectible_index]
        if dist(player_x, player_y, fx, fy) < 10:
            num_collected += 1
            collected_since_spawn += 1
            del collectibles[flashing_collectible_index]
            if collectibles:
                flashing_collectible_index = random.randint(0, len(collectibles) - 1)
            else:
                flashing_collectible_index = 0
    if collected_since_spawn >= respawn_threshold:
        spawn_new_collectibles(min(respawn_amount, max_collectibles_on_map - len(collectibles)))
        collected_since_spawn = 0

def spawn_new_collectibles(n):
    global collectibles, flashing_collectible_index
    spawned = 0
    attempts = 0
    while spawned < n and attempts < 100:
        cx = random.randint(1, cols - 2)
        cy = random.randint(1, rows - 2)
        px, py = cell_center(cx, cy)
        if grid[cx][cy] == 0:
            if dist(px, py, player_x, player_y) > 20 and all(dist(px, py, c[0], c[1]) > 10 for c in collectibles):
                collectibles.append((px, py))
                spawned += 1
        attempts += 1
    if collectibles:
        flashing_collectible_index = random.randint(0, len(collectibles) - 1)

def keyPressed():
    keys_pressed.add(key.lower())
    global current_tutorial_step, page
    if page == "InteractiveTutorial" and current_tutorial_step < len(tutorial_steps):
        step = tutorial_steps[current_tutorial_step]
        if step["key"] is None or key.lower() == step["key"]:
            current_tutorial_step += 1
            
def increase_difficulty():
    global num_enemies, difficulty_increased
    num_enemies == num_enemies*2
    time_limit = time_limit+60
    place_enemies(num_enemies)
    difficulty_increased = True
    
def keyReleased():
    keys_pressed.discard(key.lower())

def mouseClicked():
    global page, current_tutorial_step, tutorial_completed
    if page == "Welcome":
        if 50 < mouseX < 250:
            if 250 < mouseY < 300:
                setup()
                page = "Page1"
            elif 400 < mouseY < 450:
                exit()
            elif 500 < mouseY < 550:
                page = "Tutorial"
            elif 600 < mouseY < 650:
                setup()
                current_tutorial_step = 0
                tutorial_completed = False
                page = "InteractiveTutorial"
    elif page in ["Give Up", "Tutorial", "Win"]:
        page = "Welcome"
    elif page == "Give Up":
        if width // 2 - 100 < mouseX < width // 2 + 100 and height // 2 + 80 < mouseY < height // 2 + 130:
            setup()
            page = "Page1"
