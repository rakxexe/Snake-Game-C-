#include<iostream>
#include<conio.h> //for key press
#include<windows.h> //for coordinates system
#include<ctime> //for time

using namespace std;
//defining maximum length of the snake
const int max_length = 1000;

//defining directions
const char dir_up = 'W';
const char dir_down = 'S';
const char dir_left = 'A';
const char dir_right = 'D';

//defining global variables for weight and height of console
int cons_width, cons_height;

//function for initializing the screen
void initscreen(){
    HANDLE hconsole = GetStdHandle(STD_OUTPUT_HANDLE);
    CONSOLE_SCREEN_BUFFER_INFO csbi; //is predefined class and csbi is an object
    GetConsoleScreenBufferInfo(hconsole, &csbi); //is pre define function to get the coordinates 
    //setting height and width of the console
    //initally it is 1 by 1
    cons_width = csbi.srWindow.Right - csbi.srWindow.Left + 1;
    cons_height = csbi.srWindow.Bottom - csbi.srWindow.Top + 1;
}

//creating a structure for coordinates
struct point{
int coor_x, coor_y;
//creating a default constructor
point(){
}
//construvture for initializing the coordinates
point(int x, int y){
    coor_x = x;
    coor_y = y;
}    
};

//creating a class for snake
class snake{

private:
      int length;
      char direction;
public:
     //creating a multidimensional array from point structure
     point body[max_length];
     //constructor for initializing the length and direction
     snake(int x , int y){
        length = 1; //default length
        direction = dir_right; //default direction
        body[0] = point(x, y);
     }
    //creating a function for returning the length
    int getlength(){
        return length;
    }
    //creating a function for changing direction
    void change_dir(char new_dir){
        if(new_dir == dir_up && direction != dir_down){
            direction = new_dir;
    }
    else if(new_dir == dir_down && direction != dir_up){
        direction = new_dir;
    }
    else if(new_dir == dir_left && direction != dir_right){
        direction = new_dir;
    }
    else if(new_dir == dir_right && direction != dir_left){
        direction = new_dir;
    }
}
//function for moving the snake
bool move(point food){
    for(int i = length - 1; i > 0; i--){
        body[i] = body[i - 1];
    }
    int val;
    switch (direction)
    {
    case dir_up:
        val = body[0].coor_y; //for up direction y coordinate will be decreased
        body[0].coor_y = val - 1;
        break;
    case dir_down:
        val = body[0].coor_y; //for down direction y coordinate will be increased
        body[0].coor_y = val + 1;
        break;
    case dir_left:
        val = body[0].coor_x; //for left direction x coordinate will be decreased
        body[0].coor_x = val - 1;
        break;
    case dir_right:
        val = body[0].coor_x; //for right direction x coordinate will be increased
        body[0].coor_x = val + 1;
        break;
    }
    //snake bites itself
    for(int i = 1; i < length; i++){
        if(body[0].coor_x == body[i].coor_x && body[0].coor_y == body[i].coor_y){
            return false;
        }
    }
    //snake eats food
    if(body[0].coor_x == food.coor_x && body[0].coor_y == food.coor_y){
        body[length] = point(body[length - 1].coor_x, body[length - 1].coor_y);
        length++;
        }
        return true;  
}
};

//creating a class for board 
class board{

private:
       //components of the game
       snake *sna;
       const char SNAKE = 'O'; //body of the snake in game
       point food;
       const char FOOD = 'o'; //shape food in game
       int score;
public:
     //funtion for spawning food
     void spawn_food(){
        int x = rand() % cons_width; //rand is predefined function for spawning food randomly
        int y = rand() % cons_height;
        food = point(x, y);
     }
     //constructor for initializing the snake
     board(){
        spawn_food();
        sna = new snake(10,10);
        score = 0;
     }
     //destructor for deleting the snake
     ~board(){
        delete sna;
     }
     //funtion for returning score
     int getscore(){
        return score;
     }
     //funtion for setting the position
     void setpos(int x, int y){
        COORD coord; //predefined class and coord is an object
        coord.X = x;
        coord.Y = y;
        SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), coord);
     }
     //funtion for displaying score
     void displayscore(){
      setpos(cons_width / 2,0);
      cout<<"Current Score : "<<score;   
     }
     //funtion for printing snake & food
     void draw(){
        system("cls"); //predefined function to clear the screen
        //printing snake
        for(int i = 0; i < sna->getlength(); i++){
            setpos(sna->body[i].coor_x,sna->body[i].coor_y);
            cout << SNAKE;
        }
         displayscore();
        //printing food
        setpos(food.coor_x, food.coor_y);
        cout << FOOD;
     }
     //function for updating the snake & spawning food 
     bool update(){
        bool alive = sna->move(food);
        if(alive == false){
            return false;
        }
        if(food.coor_x == sna->body[0].coor_x && food.coor_y == sna->body[0].coor_y){
            score++;
            spawn_food();
        }
        return true;
     }
     //function for changing direction by key press
     void getinput(){
        if(kbhit()){
            int key = getch(); //predefined function to get key press
            if(key == 'w' || key == 'W'){
                sna->change_dir(dir_up);
            }
            else if(key == 's' || key == 'S'){
                sna->change_dir(dir_down);
            }
            else if(key == 'a' || key == 'A'){
                sna->change_dir(dir_left);
            }
            else if(key == 'd' || key == 'D'){
                sna->change_dir(dir_right);
            }
        }   
     }
};
int main(){
    srand(time(0)); //predefined function for time
    initscreen();
    board *b = new board();
    while(b->update()){
        b->getinput();
        b->draw();
        Sleep(100);
    }
     cout<<"Game over"<<endl;
    cout<<"Final score is : "<<b->getscore();
    return 0;
}
