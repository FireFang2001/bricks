练手用pygame做了一个简单到简陋的敲砖块游戏:
```python
import math
from abc import ABCMeta, abstractmethod
import pygame


class GameObject(object, metaclass=ABCMeta):
    """游戏对象类"""
    def __init__(self, x=0, y=0, color=(0, 0, 0)):
        self._x = x
        self._y = y
        self._color = color

    @property
    def x(self):
        return self._x

    @property
    def y(self):
        return self._y

    @y.setter
    def y(self, y):
        self._y = y

    @property
    def color(self):
        return self._color

    @abstractmethod
    def draw(self, screen):
        pass


class Border(GameObject):
    """边框"""
    def __init__(self, x, y, width, height, color=(255, 255, 255)):
        super().__init__(x, y, color)
        self._height = height
        self._width = width

    @property
    def height(self):
        return self._height

    @property
    def width(self):
        return self._width

    def draw(self, screen):
        pygame.draw.rect(screen, self._color, (self._x, self._y, self._width, self._height), 5)


class Brick(GameObject):
    """砖块"""
    def __init__(self, x, y, width=56, height=30, color=(125, 150, 160)):
        super().__init__(x, y, color)
        self._height = height
        self._width = width

    @property
    def height(self):
        return self._height

    @property
    def width(self):
        return self._width

    def draw(self, screen):
        pygame.draw.rect(screen, self._color,
                         (self._x, self._y, self._width, self._height), 0)
        pygame.draw.rect(screen, (50, 50, 100),
                         (self._x, self._y, self._width, self._height), 1)


class Wall(GameObject):
    """砖墙"""
    def __init__(self):
        super().__init__()
        self._bricks = []
        for i in range(5):
            for j in range(10):
                brick = Brick(60 + 56 * j, 80 + 30 * i)
                self._bricks.append(brick)

    @property
    def bricks(self):
        return self._bricks

    @bricks.setter
    def bricks(self, bricks):
        self._bricks = bricks

    def draw(self, screen):
        for brick in self._bricks:
            brick.draw(screen)


class Ball(GameObject):
    """球"""
    def __init__(self, x=340, y=605, radius=7, sx=2, sy=-4, color=(255, 255, 255)):
        GameObject.__init__(self, x, y, color)
        self._radius = radius
        self._sx = sx
        self._sy = sy

    @property
    def sx(self):
        return self._sx

    @sx.setter
    def sx(self, sx):
        self._sx = sx

    @property
    def sy(self):
        return self._sy

    @sy.setter
    def sy(self, sy):
        self._sy = sy

    @property
    def radius(self):
        return self._radius

    def move(self):
        """球的移动"""
        self._x += self._sx
        self._y += self._sy
        if (self._x + self._radius >= 640 and self._sx > 0) or \
                (self._x - self._radius <= 40 and self._sx < 0):
            self._sx = -self._sx
        if self._y - self._radius <= 40 and self._sy < 0:
            self._sy = -self._sy

    def draw(self, screen):
        pygame.draw.circle(screen, self._color,
                           (self._x, self._y), self._radius, 0)

    def distance(self, other):
        return math.sqrt((self._x - other.x) ** 2 +
                         (self._y - other.y) ** 2)

    def collide_detective(self, brick):
        """球与砖块的碰撞检测"""
        cornerx_1 = brick.x
        cornery_1 = brick.y
        cornerx_2 = brick.x + brick.width
        cornery_2 = brick.y
        cornerx_3 = brick.x + brick.width
        cornery_3 = brick.y + brick.height
        cornerx_4 = brick.x
        cornery_4 = brick.y + brick.height
        #  将一块砖分成8个部分分别判断
        if brick.y <= self._y <= (brick.y + brick.height) and \
                (brick.x - self._radius) <= self._x < brick.x:
            return 1
        elif brick.x <= self._x <= (brick.x + brick.width) and \
                (brick.y - self._radius) <= self.y < brick.y:
            return 2
        elif brick.y <= self._y <= (brick.y + brick.height) and \
                (brick.x + brick.width) < self._x <= \
                (brick.x + brick.width + self._radius):
            return 3
        elif brick.x <= self._x <= (brick.x + brick.width) and \
                (brick.y + brick.height) < self._y <= \
                (brick.y + brick.height + self._radius):
            return 4
        elif ((self._x - cornerx_1) ** 2 + (self._y - cornery_1) ** 2) \
                <= (self._radius ** 2):
            return 5
        elif ((self._x - cornerx_2) ** 2 + (self._y - cornery_2) ** 2) \
                <= (self._radius ** 2):
            return 6
        elif ((self._x - cornerx_3) ** 2 + (self._y - cornery_3) ** 2) \
                <= (self._radius ** 2):
            return 7
        elif ((self._x - cornerx_4) ** 2 + (self._y - cornery_4) ** 2) \
                <= (self._radius ** 2):
            return 8
        else:
            return 0


class Board(GameObject):
    """反弹板"""
    def __init__(self, x=290, y=615, width=100, height=5, sx=0, color=(30, 150, 170)):
        super().__init__(x, y, color)
        self._width = width
        self._height = height
        self._sx = sx

    @property
    def sx(self):
        return self._sx

    @sx.setter
    def sx(self, sx):
        self._sx = sx

    @property
    def width(self):
        return self._width

    @width.setter
    def width(self, width):
        self._width = width

    @property
    def height(self):
        return self._height

    @height.setter
    def height(self, height):
        self._height = height

    def draw(self, screen):
        pygame.draw.rect(screen, self._color,
                         (self._x, self._y, self._width, self._height), 0)

    def move(self):
        self._x += self._sx


def main():

    def collide(brick):
        """碰撞反弹"""
        num = ball.collide_detective(brick)
        if num != 0:
            wall.bricks.remove(brick)
        if num == 1 or num == 3:
            ball.sx = -ball.sx
        elif num == 2 or num == 4:
            ball.sy = -ball.sy
        elif num == 5 or num == 6 or num == 7 or num == 8:
            ball.sx = -ball.sx
            ball.sy = -ball.sy

    def refresh():
        """刷新窗口"""
        screen.fill((0, 0, 0))
        wall.draw(screen)
        ball.draw(screen)
        board.draw(screen)
        border.draw(screen)
        pygame.display.flip()

    def handle_key_event(key_event):
        """处理按键事件"""
        nonlocal start_move
        key = key_event
        if key == pygame.K_s:  # 按s键开球
            start_move = True
        elif key == pygame.K_F2:
            start_move = False
            reset_game()

    def reset_game():
        """重置游戏"""
        nonlocal ball, wall, board, game_over
        wall = Wall()
        ball = Ball()
        board = Board()
        pygame.event.clear()
        game_over = False

    clock = pygame.time.Clock()
    wall = Wall()
    ball = Ball()
    board = Board()
    border = Border(40, 40, 600, 600)
    pygame.init()
    screen = pygame.display.set_mode([680, 680])
    pygame.display.set_caption('')
    start_move = False
    running = True
    game_over = False
    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                handle_key_event(event.key)
        if not game_over:
            pressed_keys = pygame.key.get_pressed()  # 操作反弹板
            if pressed_keys[pygame.K_d] and board.x + board.width < 640:
                board.sx = 10
                board.move()
            elif pressed_keys[pygame.K_a] and board.x > 40:
                board.sx = -10
                board.move()
            if start_move:
                ball.move()
            for brick in wall.bricks:
                collide(brick)
            if board.x - ball.radius <= ball.x <= board.x + board.width + ball.radius \
                    and board.y - ball.radius <= ball.y < board.y:  # 球与板的碰撞反弹
                ball.sy = -ball.sy
                if (pressed_keys[pygame.K_d] or pressed_keys[pygame.K_a]) and \
                        (abs(ball.sx) <= 5 or ball.sx / board.sx < 0):
                    ball.sx += board.sx // 10  # 板给球的x方向的速度
            refresh()
            if ball.y > 680:
                my_font = pygame.font.SysFont('', 80)
                text = my_font.render("Game Over", 1, (255, 10, 10))
                screen.blit(text, (200, 300))
                pygame.display.flip()
                game_over = True
            if not wall.bricks:
                my_font = pygame.font.SysFont('', 80)
                text = my_font.render("You Win", 1, (255, 10, 10))
                screen.blit(text, (200, 300))
                pygame.display.flip()
                game_over = True
        clock.tick(60)
    pygame.quit()


if __name__ == '__main__':
    main()

```