---
layout: default
title:  "Python Game"
parent: My Projects
nav_order: 4
---

<h1><center>BIOL595 - Practical Biocomputing Final Project</center></h1>  

### Introduction
--------------------------------------------------------------------------------
For my final project in the BIOL595 course at Purdue, I coded a Python game using object oriented programming. The package/library <a href="https://www.pygame.org/news" target="_blank">Pygame</a> was used to make coding of the game easier. Though the game cannot be run on this site, I have included .gifs and images to showcase some of the art that I have drawn for the game. Some of the Python codes are also shown.

### Digital Drawing Tools
--------------------------------------------------------------------------------
<a href="https://firealpaca.com/" target="_blank">FireAlpaca</a>, a free Digital Painting Software, and the Wacom Intuos tablet were used for all the digital arts for this project.

<figure>
<img src="/assets/img/biol595proj_img/Screenshot (259).png" alt="Trulli" style="width:100%">
</figure>

### Cut Screens - Story
--------------------------------------------------------------------------------
When the player interacts with an NPC, a cutscene like the one below would play. These cutscences are composed of multiple images, which get cycled through as the placer presses the `space` bar. A class named _CutScene_ when called creates an object that iterates through every image in a given folder and displays them one by one after `space` is pressed. 

<figure>
<img src="/assets/img/biol595proj_img/ezgif-5-d847112642.gif" alt="Trulli" style="width:100%">
</figure>


```python
class CutScene(Screen):
    def __init__(self, img_path):
        super().__init__()
        self.sprite_list = []  # list of sprites to be updated/drawn onto the screen
        self.name_list = os.listdir(img_path)  # list of file names - should be in order
        self.img_path = img_path

    def run_screen(self):
        self.is_running = True
        # begin game/window loop
        for scene_name in self.name_list:
            scene = pg.image.load((self.img_path + "/" + scene_name))  # load in the scene image
            self.game_screen.blit(scene, (0, 0))
            self.update(self.sprite_list)
            pg.time.Clock().tick(2)  # This decreases sensitivity (1 space bar does not skip several scenes)
            while self.is_running:
                self.check_events()
                if pg.key.get_pressed()[K_SPACE]:
                    break
                self.draw(self.sprite_list)
```

### Sketches and Ideas
--------------------------------------------------------------------------------
Below are some sketches that did not make it into the final game. 

<figure>
<img src="/assets/img/biol595proj_img/bacteriophage.png" alt="Trulli" style="width:100%">
</figure>

<img src="/assets/img/biol595proj_img/ebola.png" style="float: left; width: 47%; margin-right: 1%; margin-bottom: 0.5em;">
<img src="/assets/img/biol595proj_img/ampullavirus.png" style="float: left; width: 47%; margin-right: 1%; margin-bottom: 0.5em;">
<p style="clear: both;">

<img src="/assets/img/biol595proj_img/screen2-background.png" style="float: left; width: 47%; margin-right: 1%; margin-bottom: 0.5em;">
<img src="/assets/img/biol595proj_img/screen2-heart_bg.png" style="float: left; width: 47%; margin-right: 1%; margin-bottom: 0.5em;">
<p style="clear: both;">

<figure>
<img src="/assets/img/biol595proj_img/screen2-lungs_bg.png" alt="Trulli" style="width:100%">
</figure>
<p>&nbsp;</p>


### main.py
----------------------------------------------------------------------
The game was split into 3 Python scripts - main.py, sprites.py, and settings.py - for better organization of the codes. main.py consists of parent classes such as _Game_ and _Screen_, along with multiple child/subclasses of _Screen_ such as _CombatScreen_, _MapScreen_, etc.  
Again, the codes cannot be run on this static site, but they are still presented below. 


```python
import pygame as pg
import sys
from settings import *
from sprites import *


class Game:
    def __init__(self):
        pg.init()  # initialize the game - need this for Pygame to work
        pg.display.set_caption(GAME_TITLE)  # game window name/caption
        pg.event.set_allowed([pg.QUIT, pg.KEYDOWN, pg.KEYUP])  # only allow for certain events to be detected
        # pg.display.set_icon('path_to_image')  # set the window icon image (instead of the ugly pygame snake)

    def run_game(self):
        """
        Create the appropriate Screen instance for each screen that will be run in the game, then call the function
        to run each of those screens/window.
        """
        # START SCREEN:
        self.screen_start = StartScreen()
        self.screen_start.run_screen()
        # MAP SCREEN:
        self.screen_map = MapScreen()
        self.screen_map.run_screen()
        # CUTSCENE (dialogues) - none at the moment
        # self.screen_cutscene = CutScene()
        # self.screen_cutscene.run_screen()
        # COMBAT SCREEN:
        self.screen_combat = CombatScreen()
        self.screen_combat.run_screen()
        # ENDING SCREEN:
        self.screen_end = EndScreen()
        self.screen_end.run_screen()


class Screen:
    def __init__(self):
        """
        Parent class of all the child screen objects.
        """
        # get information for the game window:
        self.width = SCREEN_WIDTH
        self.height = SCREEN_HEIGHT
        self.game_screen = pg.display.set_mode((self.width, self.height))  # create game screen/window
        self.is_running = True  # keep track of if the screen is running or not - when initialized = running
        self.sprite_groups = SpriteGroups()  # create all the groups for different sprites to be used in the screens

        # PLAYER:
        # create player - only 1 in the entire game (between all the screens)
        self.player = Player(self)  # create instance of Player class
        self.lives_img = pg.image.load(path.join(DIR_SPRITES, 'pete_head.png')).convert_alpha()  # set image of player

        # FONT - does not work at the moment
        self.font_obj = pg.font.Font(path.join(DIR_CURRENT, 'pixel_font.TTF'), 2)

    def load_image(self, img_path, img_name):
        """
        Loads in an image (for the background).
        Parameters: img_path = path to image file; img_name = name of image file
        """
        self.img = pg.image.load(path.join(img_path, img_name)).convert()

    def check_events(self):
        """
        Check for the events
        """
        for event in pg.event.get():
            if event.type == QUIT:
                pg.quit()
                sys.exit()

    def update(self, sprite_list=[]):
        """
        Updates all the sprites specified (each sprite has a different update() function).
            - Depending on the condition, sprites might: move, disappear, etc.
        Parameters: sprite_list = list of sprite groups to be updated, sprite groups contain different
            classes/types of sprite objects.
        """
        for group in sprite_list:  # update all the sprites given
            for sprite in group:
                sprite.update()

    def draw(self, sprite_list=[], player_stat_dis=True):
        """
        For each sprite in the groups, draw them on the screen, then display the player's status (health, lives).
        Lastly, the screen containing all the drawn on images will be displayed(flip()).
        Parameters: sprite_list: list of groups of sprites for that particular screen; player_stat_dis: True=display
            the player sprite, False=does not display player sprite.
        """
        # DRAW SPRITES:
        for group in sprite_list:
            for sprite in group:
                self.game_screen.blit(sprite.image, sprite.rect)

        # DISPLAY PLAYER STATUS:
        if player_stat_dis:
            self.display_hp(self.player.health)
            self.display_lives()
        pg.display.flip()

    def display_hp(self, player_health):
        """
        Displays the health of the player, decreases over time when the player gets bit by enemy bullet
        Parameter: player_health: health of the player
        """
        if player_health < 0:
            player_health = 0  # this makes it so that the health bar doesn't become negative

        # defining the entire health bar box:
        hp_bar_width = 5.75*TILE_LEN
        hp_bar_height = 0.95*TILE_LEN
        x = 2*TILE_LEN
        y = 1*TILE_LEN

        # using the player's current health percentage as width of the health bar:
        current_hp_len = (player_health / PLAYER_HEALTH) * hp_bar_width

        # draw bars on the screen:
        if (player_health/PLAYER_HEALTH) > 0.75:
            pg.draw.rect(self.game_screen, GREEN, pg.Rect(x, y, current_hp_len, hp_bar_height), 0, 5)
        elif (player_health/PLAYER_HEALTH) < 0.25:
            pg.draw.rect(self.game_screen, RED, pg.Rect(x, y, current_hp_len, hp_bar_height), 0, 5)
        else:
            pg.draw.rect(self.game_screen, YELLOW, pg.Rect(x, y, current_hp_len, hp_bar_height), 0, 5)
        pg.draw.rect(self.game_screen, BLACK, pg.Rect(x, y, hp_bar_width, hp_bar_height), 2, 5)

    def display_lives(self):
        """
        Display the player's remaining lives.
        """
        # Display the number of lives the player has:
        live_img = self.lives_img
        live_img_rect = live_img.get_rect()
        live_img_rect.y = 0.5 * TILE_LEN
        for live in range(self.player.lives):
            live_img_rect.x = (20*TILE_LEN) + (37*live)  # space out the lives
            self.game_screen.blit(live_img, live_img_rect)


class StartScreen(Screen):
    def __init__(self):
        super().__init__()
        self.load_image(DIR_SCREENS, 'start_screen.png')  # load in the start screen image

    def run_screen(self):
        self.is_running = True
        # begin game/window loop
        self.game_screen.blit(self.img, (0, 0))  # display start image on screen
        while self.is_running:
            if pg.key.get_pressed()[K_SPACE]:  # if the player presses space, end the loop - move onto next screen
                self.is_running = False
            self.check_events()  # check for events
            self.update()
            self.draw(player_stat_dis=False)  # does not display player on start screen
            pg.time.wait(100)  # the game waits for a little before repeating the loop


class MapScreen(Screen):
    """
    MapScreen objects display the player, NPCs, and the tile-based map as the background. The player is able to walk
    within the pre-determined path and interact with the NPC
    """
    def __init__(self):
        super().__init__()
        # list of sprite groups to be displayed on this screen:
        self.sprite_list = [self.sprite_groups.player_group, self.sprite_groups.friendly_NPCs,
                            self.sprite_groups.walls]
        # create 2 NPC sprite to place on the map:
        self.friendlyNPC1 = FriendlyNPC(self, "frog.png", (5.5*TILE_LEN, 12.5*TILE_LEN))

        # load in the map:
        self.map = SimpleMap(path.join(DIR_CURRENT, 'map_tiles.txt'), self)  # determines where walls are placed
        self.load_image(DIR_SCREENS, 'main_map.png')  # load in the background image

    def kill_sprites(self):
        pass

    def run_screen(self):
        self.is_running = True
        # begin game/window loop
        while self.is_running:
            self.check_events()  # check for events
            self.game_screen.blit(self.img, (0, 0))  # display the background image
            self.update(self.sprite_list)  # update all the sprites in given list of groups
            self.draw(self.sprite_list)  # draw all the sprites on the screen
            pg.time.wait(MAP_SCR_WAIT)
        self.kill_sprites()

# commented out for now
# class CutScene(Screen):
#     def __init__(self):
#         super().__init__()
#
#     def run_screen(self):
#         self.game_screen.fill(GREEN)  # TEMPORARY screen color - delete later
#         self.is_running = True
#         # begin game/window loop
#         while self.is_running:
#             self.check_events()
#             self.temp_next_screen()
#             self.update()
#             self.draw()


class CombatScreen(Screen):
    def __init__(self):
        super().__init__()
        # list of sprite group that will be displayed:
        self.sprite_list = [self.sprite_groups.player_group, self.sprite_groups.hostile_NPCs,
                            self.sprite_groups.bullets, self.sprite_groups.walls]
        self.hostileNPC1 = HostileNPC(self, "enemy.png", (23*TILE_LEN, 9*TILE_LEN))  # create an enemy NPC

        # change player attributes:
        self.player.rect.center = (4*TILE_LEN, 9*TILE_LEN)  # place the player somewhere else on screen
        self.player.constraint = COMBAT_SCREEN_CONSTRAINT  # allow player to only move in the specified region
        del_list = ['w', 'a', 's']  # change the player's animation (w,a,s will not change player image direction)
        for key in del_list:
            self.player.allowed_img_change[key]=False

        # load in the map:
        self.map = SimpleMap(path.join(DIR_CURRENT, 'combat_tiles.txt'), self)
        self.load_image(DIR_SCREENS, 'combat_map.png')

    def run_screen(self):
        self.is_running = True
        # begin game/window loop
        while self.is_running:
            self.check_events()
            self.update(self.sprite_list)
            self.game_screen.blit(self.img, (0,0))
            self.draw(self.sprite_list)
            pg.time.wait(5)


class EndScreen(Screen):
    def __init__(self):
        super().__init__()
        # different screens will display depending on whether the player died, or the player killed enemy successfully
        if self.player.lives <= 0:
            name = 'game_over.png'
        else:
            name = 'win.png'
        self.load_image(DIR_SCREENS, name)

    def run_screen(self):
        self.is_running = True
        self.game_screen.blit(self.img, (0, 0))
        # begin game/window loop
        while self.is_running:
            self.check_events()
            self.update()
            self.draw(player_stat_dis=False)


class SpriteGroups:
    def __init__(self):
        """
        Function that creates all the sprite groups needed for the game
        """
        # initialize sprite groups
        self.player_group = pg.sprite.Group()  # group with just player - so that player can be iterated with others
        self.all_sprites = pg.sprite.Group()  # group of all the sprites
        self.NPCs = pg.sprite.Group()  # group of all NPC - MIGHT NOT NEED THIS
        self.hostile_NPCs = pg.sprite.Group()  # group of all the hostile NPC 
        self.friendly_NPCs = pg.sprite.Group()  # group of all the friendly NPCs
        self.walls = pg.sprite.Group()  # group of all the wall sprites
        self.bullets = pg.sprite.Group()  # group of all the bullets - MIGHT NOT NEED THIS
        self.player_bullets = pg.sprite.Group()  # group of all the player's bullets
        self.hostile_bullets = pg.sprite.Group()  # group of all the enemies' bullets

    def create_new_group(self, group_name):
        pass


# NOTE: not sure if this needs to be its own class, maybe just a function instead
class SimpleMap:
    def __init__(self, fname, screen):
        self.lines = []  # list to hold each line in the file
        self.fname = fname
        self.open_file()
        self.read_file()
        self.make_map(screen)

    def open_file(self):
        try:
            self.file = open(self.fname, 'rt')
        except OSError:
            print(f'Error opening file ({self.fname})')
            # since you cannot continue, exit with status = 1
            exit(1)

    def read_file(self):
        for line in self.file:
            self.lines.append(line.strip())  # add each line of the file into a list

    def make_map(self, screen):
        """
        Creates a "map" - places transparent square objects on the map so that the player cannot pass through them.
        Parameter: scree: the screen for the walls to be placed on
        """
        for row, line in enumerate(self.lines):  # for each line in the file, get its row# (this will be x position)
            for col, map_value in enumerate(line):  # for each line, get the column# (this is the y position)
                if map_value == '0':
                    Wall(screen, col, row)  # create new wall at position specified in the .txt. document

if __name__ == '__main__':
    GAME = Game()  # create the game
    GAME.run_game()  # run the game
```

### sprites.py
----------------------------------------------------------------------
This Python scripts contains classes for creating and manipulating sprites, which are images that represent game assets, such as player characters, npcs, enemies, projectiles, etc. Sprite manipuation includes collision detection, movement, updating animation, etc. 


```python
import pygame as pg
import random
from pygame.locals import *
from settings import *


def add_to_group(sprite_obj, group_list):
    """
    Add sprite object to the groups given in a list
    Parameters: sprite_obj: the sprite object to add to a sprite group; group_list: list of sprite groups
        to add the sprite object to
    """
    for group in group_list:
        group.add(sprite_obj)


class Sprite(pg.sprite.Sprite):
    """
    Main sprite class, all other sprite classes below are child classes of this class. This class is a child class of
     Pygame's sprite object.
    """
    def __init__(self, screen):
        super().__init__()
        self.sprite_groups = screen.sprite_groups  # get the sprite groups from the main screen
        self.screen = screen  # get the screen (for things to be displayed on)


class Player(Sprite):
    def __init__(self, screen):
        super().__init__(screen)
        # LOAD IMAGES (different directions):
        self.w_image = pg.image.load(path.join(DIR_SPRITES,'t_player_w.png')).convert_alpha()
        self.a_image = pg.image.load(path.join(DIR_SPRITES,'t_player_a.png')).convert_alpha()
        self.s_image = pg.image.load(path.join(DIR_SPRITES,'t_player_s.png')).convert_alpha()
        self.d_image = pg.image.load(path.join(DIR_SPRITES,'t_player_d.png')).convert_alpha()

        # set player image + rect:
        self.image = self.w_image
        self.rect = self.image.get_rect()
        # since the map is made out of small square tiles, (x,y) are multiplied by tile_len to place at desired tile
        self.rect.center = (6 * TILE_LEN, 6.5 * TILE_LEN)

        # PLAYER GROUPS - add player to appropriate sprite groups:
        self.groups = [self.sprite_groups.all_sprites, self.sprite_groups.player_group]
        add_to_group(self, self.groups)

        # PLAYER MOVEMENT SETTINGS:
        self.dx = 0  # movement in the x direction
        self.dy = 0 # movement in the y direction
        self.constraint = WHOLE_SCREEN_CONSTRAINT  # region where player can move
        self.allowed_img_change = {'w': True, 'a': True, 's': True, 'd': True}  # animation directions that are allowed

        # PLAYER STATS - what the player starts with:
        self.health = PLAYER_HEALTH
        self.lives = PLAYER_LIVES

        # TIMING:
        # used to keep track of how much time has passed (before the player does an action)
        self.prev_time = pg.time.get_ticks()

    def get_movements(self):
        """
        Change coordinates of the sprite, and create bullets based on input keys from the player.
        """
        self.dx, self.dy = 0, 0  # set at 0 so that sprite stops moving when key isn't pressed

        # change coordinates and also sprite animation when certain keys are pressed:
        key_inputs = self.key_inputs
        if self.rect.top > self.constraint['top']:
            if key_inputs[K_w] or key_inputs[K_UP]:
                self.dy = -4
                if self.allowed_img_change['w']:
                    self.image = self.w_image

        if self.rect.left > self.constraint['left']:
            if key_inputs[K_a] or key_inputs[K_LEFT]:
                self.dx = -4
                if self.allowed_img_change['a']:
                    self.image = self.a_image

        if self.rect.bottom < self.constraint['bottom']:
            if key_inputs[K_s] or key_inputs[K_DOWN]:
                self.dy = 4
                if self.allowed_img_change['s']:
                    self.image = self.s_image

        if self.rect.right < self.constraint['right']:
            if key_inputs[K_d] or key_inputs[K_RIGHT]:
                self.dx = 4
                if self.allowed_img_change['d']:
                    self.image = self.d_image

        if key_inputs[pg.K_SPACE]:
            self.shoot()  # shoot bullet when player presses space

    def npc_collide(self):
        """
        Detect collision with NPCs, allow for interaction with NPC,
         and ensure that player does not run through the NPC.
        """
        # if player collides into friendly NPC:
        if pg.sprite.spritecollide(self, self.sprite_groups.NPCs, False):  # False: do not remove NPC sprite
            if self.key_inputs[K_SPACE]:  # takes the player to another screen
                pg.time.wait(10)
                self.screen.is_running = False
            # make it so that the player does not go through the NPC sprite image
            self.rect.x -= self.dx
            self.rect.y -= self.dy

    def walls_collide(self):
        """
        Detect collision with walls and make sure player does not go through objects in the sprite group walls
        """
        collision = pg.sprite.spritecollide(self, self.sprite_groups.walls, False)
        if collision:
            # change direction of movement so player does not run through the walls
            self.rect.x -= self.dx
            self.rect.y -= self.dy

            # NOTE: in the code below, I attempted to make it so that when the player moves diagonally
            # into a wall, the sprite slides against the wall instead of stopping completely - does not work for now
            # wall = collision[0]
            # if self.dx > 0:  # dx>0 means sprite is moving right, which means it is hitting the left edge of wall
            #     self.rect.right = wall.rect.left - 1  # set the right of the player back to the left of the wall
            # if self.dx < 0:  # sprite is moving left and is hitting right of wall
            #     self.rect.left = wall.rect.right + 1
            #
            # self.rect.x -= 0
            #
            # if self.dy > 0:  # sprite moving down = hitting the top of wall
            #     self.rect.bottom = wall.rect.top - 1
            # if self.dy < 0:  # sprite moving up and is hitting bottom of wall
            #     self.rect.top = wall.rect.bottom + 1

            # self.rect.y -= 0

    def bullets_collide(self):
        """
        Detection for collision with bullets, then adjust the player's health and lives accordingly.
        """
        if pg.sprite.spritecollide(self, self.sprite_groups.hostile_bullets, True):  # True: remove bullet sprite
            self.health -= H_NPC_DAMAGE
        if self.health <= 0:
            self.lives -= 1  # decrease player's lives by 1 when they run out of health
            self.health = PLAYER_HEALTH  # reset player's health to 100%
        if self.lives <= 0:
            self.screen.is_running = False  # ends the window/screen when the player runs out of lives

    def shoot(self):
        """
        Create bullets in interval when the function is called
        """
        time_now = pg.time.get_ticks()  # get the time it is right now
        time_passed = time_now - self.prev_time  # get the time it has been since the last recorded time
        if time_passed > 100:  # if a certain amount of time has passed, shoot another bullet
            bullet = PlayerBullets(self.screen, GREEN, self.rect.right, self.rect.centery)
            self.prev_time = time_now  # re-set the time

    def update(self):
        """
        Update the sprite based on: key inputs from user, collision with NPC, and collision with wall.
        """
        self.key_inputs = pg.key.get_pressed()
        self.get_movements()  # get the key presses and corresponding movements that should be made

        self.rect.x += self.dx  # change the x position based on change in dx
        self.rect.y += self.dy  # change y position based on dy

        self.npc_collide()  # check to see if player collides with any NPCs
        self.walls_collide()  # check to see if player collides with any wall
        self.bullets_collide()  # see if any bullet hits the player


class NPC(Sprite):
    def __init__(self, screen, img_name, center):
        super().__init__(screen)
        self.image = pg.image.load(path.join(DIR_SPRITES, img_name)).convert_alpha()
        self.rect = self.image.get_rect()
        self.rect.center = center  # place NPC at chosen location on screen


class HostileNPC(NPC):
    def __init__(self, screen, img_name, center):
        super().__init__(screen, img_name, center)
        # GROUPS - add NPC to appropriate groups:
        self.groups = [self.sprite_groups.all_sprites, self.sprite_groups.NPCs,
                       self.sprite_groups.hostile_NPCs]
        add_to_group(self, self.groups)
        self.health = H_NPC_HEALTH

        # TIMING (for random y movement - the NPC will be moving randomly in the y direction) :
        self.prev_time = pg.time.get_ticks()
        self.rand_dy()  # randomly set the dy movement

    def rand_dy(self):
        """
        Randomizes sprite's movement in the y direction (up/down, how fast/slow)
        """
        self.dy = random.randrange(-3, 5, 2)

    def randomize_move(self):
        """
        Randomizes the movement of the sprite after the specified amount of time has passed
        """
        time_now = pg.time.get_ticks()  # get the time it is right now
        time_passed = time_now - self.prev_time  # get the time it has been since the last time
        if time_passed > 200:  # if a certain amount of time has passed
            self.rand_dy()  # randomize the y speed+direction again
            self.prev_time = time_now  # re-set the time

    def walls_collide(self):
        """
        Change the sprite's direction of movement to be the opposite when there is a collision with the walls - so
         that the sprite does not keep trying to move up and stay below/above the wall edge ( which makes it easier
         for the players).
        """
        if pg.sprite.spritecollide(self, self.sprite_groups.walls, False):
            self.rect.y -= self.dy  # change y direction if collide with walls

    def bullets_collide(self):
        """
        Change the sprite's health when it collides with player's bullets
        """
        if pg.sprite.spritecollide(self, self.sprite_groups.player_bullets, True):  # True: delete bullets
            self.health -= PLAYER_DAMAGE

    def shoot_bullets(self, bullet_num):
        """
        Create specified number of bullet objects when called
        Parameters: bullet_num: number of bullets to create
        """
        for i in range(bullet_num):
            bullet = HostileBullets(self.screen, BLACK, self.rect.left, self.rect.centery)

    def display_health(self):
        """
        NOTE - does not work for now
        My attempt at displaying the sprite's health (in number) underneath its image
        """
        health_text = self.screen.font_obj.render(str(self.health), BLACK, WHITE)
        health_text_rect = health_text.get_rect()
        # place health text under NPC:
        health_text_rect.centerx = self.rect.centerx
        health_text_rect.top = self.rect.bottom + 5
        self.screen.game_screen.blit(health_text, health_text_rect)

    def update(self):
        self.randomize_move()  # randomize the sprite's movement
        self.rect.y += self.dy  # change the position on the screen based on the movement above
        self.walls_collide()  # check to see if sprite hits borders - make sure it doesn't go past the border
        self.bullets_collide()  # see if player's bullets hit NPC

        # shoot bullets randomly:
        if random.randrange(1, H_NPC_BULLET_RATE) == 5:
            self.shoot_bullets(H_NPC_BULLET_NUM)
        if self.health <= 0:
            self.screen.is_running = False  # end the combat screen when the NPC 'dies'


class FriendlyNPC(NPC):
    def __init__(self, screen, img_name, center):
        super().__init__(screen, img_name, center)

        # FRIENDLY NPC GROUPS - add NPC to appropriate groups: 
        self.groups = [self.sprite_groups.all_sprites, self.sprite_groups.NPCs,
                       self.sprite_groups.friendly_NPCs]
        add_to_group(self, self.groups)

    def update(self):
        # does not move - for now
        pass


class Wall(Sprite):
    def __init__(self, screen, x, y):
        """
        Wall objects are transparent squares/tiles. There are 2 reasons for this:
            1. So that the background images can be seen
            2. When there are many images that are used as walls, using the background as the wall image, and the
             invisible/transparent object to block player movement, we don't have to load in many images
        """
        super().__init__(screen)
        self.image = pg.image.load(path.join(DIR_SPRITES, 'transparent.png')).convert_alpha()
        self.rect = self.image.get_rect()

        # WALL GROUPS - add wall to appropriate groups:
        self.groups = [self.sprite_groups.all_sprites, self.sprite_groups.walls]
        add_to_group(self, self.groups)

        # Place the wall at the specified position
        self.rect.x = x * TILE_LEN
        self.rect.y = y * TILE_LEN


class Bullets(Sprite):
    def __init__(self, screen, color, x, y):
        super().__init__(screen)
        # TEMPORARY BULLET SHAPES - until I have an image to add in:
        self.image = pg.Surface((8, 8))
        self.rect = self.image.get_rect()
        self.image.fill(color)

        self.rect.centery = y  # middle of bullet y = middle of "host's" y

    def hit_wall(self):
        if pg.sprite.spritecollide(self, self.sprite_groups.walls, False):
            self.kill()  # remove bullet from group (it disappears from screen)

    def hit_target(self, target, target_result):
        """
        Check to see if bullet hits another sprite object, then, depending on the argument given, the sprite will
         either be deleted, or not.
        Parameters:
        target: target sprite for collision detection (ex: enemy's bullets)
        target_result:
            - True = the sprite will be deleted
            - False = the sprite will not be deleted
        """
        # delete target when bullet hits it:
        if pg.sprite.spritecollide(self, target, target_result):
            self.kill()


class PlayerBullets(Bullets):
    def __init__(self, screen, color, x, y):
        super().__init__(screen, color, x, y)
        self.rect.left = x  # bullet's left = player's right (bullet comes out of player from the right)
        self.dx = PLAYER_BULLET_SPEED  # speed of bullet (only moves to the right)

        # GROUPS - add bullets to groups:
        self.groups = [self.sprite_groups.all_sprites, self.sprite_groups.bullets,
                       self.sprite_groups.player_bullets]
        add_to_group(self, self.groups)

    def update(self):
        self.rect.left += self.dx  # move the bullet to the right every update
        self.hit_wall()  # remove bullet when it hits the wall
        self.hit_target(self.sprite_groups.hostile_bullets, True)  # delete both bullets when they hit each other


class HostileBullets(Bullets):
    def __init__(self, screen, color, x, y):
        super().__init__(screen, color, x, y)
        self.rect.right = x  # bullet's right = enemy's left (bullet comes out of enemies from the left)
        self.dx = H_NPC_BULLET_SPEED  # bullet's speed (moves left)
        self.dy = random.randrange(-1, 1)  # randomize enemy's bullet horizontal (y) directions

        # GROUPS - add bullets to groups:
        self.groups = [self.sprite_groups.all_sprites, self.sprite_groups.bullets,
                       self.sprite_groups.hostile_bullets]
        add_to_group(self, self.groups)

    def update(self):
        self.rect.right += self.dx
        self.rect.centery += self.dy
        self.hit_wall()  # check to see if bullets hit the wall
```

### settings.py
-------------------------------------------------------------
Lastly, the settings script mostly contains variables that can the edited to change the game, for example, character speed. All the settings are placed in one single script so that they can easily be found and changed (compared to being in main.py or sprites.py where they can be drowned out by the many lines of code).


```python
# IMPORTS
from os import path

# TEMPORARY SETTINGS - delete later:
# COLORS:
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (179, 96, 96)
GREEN = (96, 179, 118)
BLUE = (96, 126, 179)
YELLOW = (179, 175, 96)
PURPLE = (154, 96, 179)

# GAME settings:
GAME_TITLE = "BIOL595 PROJECT"
SCREEN_WIDTH = 832  # 26 x 18
SCREEN_HEIGHT = 576
TILE_LEN = 32

# DIRECTORY PATHS settings - paths to common directories:
DIR_CURRENT = path.dirname(__file__)  # get the path to the current folder of the game
DIR_SCREENS = path.join(DIR_CURRENT, "Screens")
DIR_SPRITES = path.join(DIR_CURRENT, "Sprites")

# start screen settings: none at the moment

# main map settings:
MAP_SCR_WAIT = 10

# cutscene settings - none at the moment

# combat screen settings:
COMBAT_SCR_WAIT = 1

# SPRITE SETTINGS:
# Player settings:
WHOLE_SCREEN_CONSTRAINT = {"left": 0, "right": SCREEN_WIDTH, "top": 0, "bottom": SCREEN_HEIGHT}
COMBAT_SCREEN_CONSTRAINT = {"left": 0, "right": 7*TILE_LEN, "top": 0, "bottom": SCREEN_HEIGHT}
PLAYER_HEALTH = 100
PLAYER_LIVES = 4
PLAYER_DAMAGE = 15  # how much damage the player's bullets affect the enemy
PLAYER_BULLET_SPEED = 3  # how fast the bullet moves, has to be positive (to move to the right of screen)

# Hostile NPC settings:
H_NPC_HEALTH = 200  # health of the hostile NPC
H_NPC_DAMAGE = 30  # how much damage the bullets deal on player's health
H_NPC_BULLET_NUM = 3  # number of bullets at a time
H_NPC_BULLET_RATE = 35  # how fast the NPC shoots the bullets (smaller number = faster | has to be >5)
H_NPC_BULLET_SPEED = -3  # how fast bullet moves - has to be negative
```
