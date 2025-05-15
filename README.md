# shinchan-opengl-animation
An OpenGL-based animated project recreating Shinchan with interactive controls using C++.
#include <windows.h>
#include <GL/glut.h>
#include <math.h>
#include <fstream>
#include "imageloader.h"
#include <cstdlib>
#include <ctime>
#include <cmath>
#include <vector>
#ifndef M_PI
#define M_PI 3.14159265358979323846
#endif


//angle of rotation for the direction
float angle = 0.0f;

//heartbeat benjol var
float scale = 1.0;
float delta = 0.01f; //speed
bool increasing = true;


//actual vector representing the camera direction
float lx = 0.0f, lz = -1.0f;

//XZ position of the camera
float x = 0.0f, z = 5.0f;

float rotationspeed = 0.1f;

//the key states. There variables will be zero
//when no key is being presses
float deltaAngle = 0.0f;
float deltaMove = 0;
float deltaStrafe = 0;
int xOrigin = -1;

//================================SHAKE/DANCE========================================
// Head shaking animation variables
bool shakingHead = false;
float shakeAngle = 0.0f;
float shakeSpeed = 4.0f; // degrees per frame
float shakeTime = 0.0f;
const float shakeDuration = 0.7f; // seconds

//================================LIE DOWN========================================
bool LieDown = false; // Flag to start the standing up animation
bool LieDownComplete = false; // Flag to check if standing animation is complete
float rotationAngle = 0.0f; // Current rotation angle for standing up
float TranslatePosition = 0.0f;

const float TranslateSpeed = 0.15f;
const float LieDownSpeed = 5.0f; // Rotation speed in degrees per frame
const float targetAngle = -80.0f; // Target rotation angle for standing up
const float targetPosition = -2.4f;

//LEGS
bool LieDown1 = false;
bool LieDownComplete1 = false;
float TranslatePosition1 = 0.0f;
const float TranslateSpeed1 = 3.0f;
const float targetPosition1 = 0.5f;

// LEFT HAND
bool LieDown2 = false; // Flag to start the lie down animation
bool LieDownComplete2 = false; // Flag to check if standing animation is complete
float rotationAngle2 = 0.0f;
float TranslatePosition2 = 0.0f;
float TranslatePosition6 = 0.0f;
float rotationAngle5 = 0.0f;
float TranslatePosition7 = 0.0f;

const float TranslateSpeed7 = 0.1; //-ve
const float TranslateSpeed6 = 0.1; //-ve
const float LieDownSpeed5 = 60.0f;
const float TranslateSpeed2 = 0.1f;//+ve
const float LieDownSpeed2 = 60.0f; // Rotation speed in degrees per frame

const float targetAngle2 = -60.0f; // Target rotation angle for lie down
const float targetPosition2 = 0.7; //xaxis
const float targetAngle5 = -50.0f;
const float targetPosition6 = -1.0f; //y
const float targetPosition7 = -1.5f; //z

// RIGHT HAND
bool LieDown3 = false;
bool LieDownComplete3 = false;
float rotationAngle3 = 0.0f;
float TranslatePosition4 = 0.0f;
float rotationAngle4 = 0.0f;
float TranslatePosition5 = 0.0f;
float rotationAngle6 = 0.0f;

const float LieDownSpeed6 = 40.0f;
const float TranslateSpeed5 = 0.1f;
const float LieDownSpeed4 = 40.0f;
const float TranslateSpeed4 = 0.1f;
const float LieDownSpeed3 = 40.0f;

const float targetAngle3 = -60.0f; //x-angle
const float targetPosition4 = -2.8f;
const float targetAngle4 = 67.0f;
const float targetPosition5 = 1.0f;
const float targetAngle6 = -10;


//================================================================================


void changeSize(int w, int h)
{
    //Prevent a divide by zero, when window is too short
    //(you cant make a window of zero width)

    if (h == 0)
        h = 1;

    float ratio = w * 1.0 / h;

    //use the projection matrix
    glMatrixMode(GL_PROJECTION);

    //reset matrix
    glLoadIdentity();

    //Set the viewport to be the entire window
    glViewport(0, 0, w, h);

    //set the correct perspective
    gluPerspective(45.0f, ratio, 0.1f, 100.0f);

    //Get Back to the Modelview
    glMatrixMode(GL_MODELVIEW);

}



void computePos(float deltaMove,float deltaStrafe)
{
    x += deltaMove * lx * 0.1f;
    z += deltaMove * lz * 0.1f;

    float strafeX = -lz;
    float strafeZ = lx;

    x += deltaStrafe * strafeX * 0.1f;
    z += deltaStrafe * strafeZ * 0.1f;

}

void drawCircle(float radius, int segments) {
    glBegin(GL_POLYGON);
    for (int i = 0; i < segments; i++) {
        float theta = 2.0f * 3.1415926f * float(i) / float(segments); // Get the current angle
        float x = radius * cosf(theta); // Calculate the x component
        float y = radius * sinf(theta); // Calculate the y component
        glVertex2f(x, y); // Output vertex
    }
    glEnd();
}

void drawFace()
{
    // Set material properties for the face
    GLfloat matAmbient[] = { 1.0f, 0.831f, 0.718f, 1.0f };
    GLfloat matDiffuse[] = { 1.0f, 0.831f, 0.718f, 1.0f };
    GLfloat matSpecular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    GLfloat matShininess[] = { 50.0f };

    glMaterialfv(GL_FRONT, GL_AMBIENT, matAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, matDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, matSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, matShininess);

    glColor3f(1.0f, 0.831f, 0.718f);

    //draw left cheek
    glPushMatrix();
    glTranslatef(0.1, 0.1, 0.5);
    glutSolidSphere(0.8f, 100, 100); //size 
    glPopMatrix();

    //draw right cheek
    glPushMatrix();
    glTranslatef(2.15f, 0.1f, 0.5f);
    glutSolidSphere(0.8f, 100, 100);
    glPopMatrix();

    glPushMatrix(); //dagu
    glTranslatef(1.20f, 0.0f, 0.0f);
    glutSolidSphere(0.55f, 100, 100);
    glPopMatrix();

    glPushMatrix(); //benjol
    glTranslatef(2, 3.1, -0.75);
    glScalef(scale, scale, scale);
    glutSolidSphere(0.4, 100, 100);
    glPopMatrix();

    glTranslatef(1.27f, 1.0f, -0.5f);
    glutSolidSphere(1.55f, 100, 100); //face

    // Set material properties for the eyebrow
    GLfloat browMatAmbient[] = { 0.0f, 0.0f, 0.0f, 1.0f }; // Black eyebrow color
    GLfloat browMatDiffuse[] = { 0.0f, 0.0f, 0.0f, 1.0f }; // Black eyebrow color
    GLfloat browMatSpecular[] = { 0.0f, 0.0f, 0.0f, 1.0f }; // No specular highlights for eyebrow
    GLfloat browMatShininess[] = { 0.0f }; // No shininess for eyebrow

    glMaterialfv(GL_FRONT, GL_AMBIENT, browMatAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, browMatDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, browMatSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, browMatShininess);

    //eyebrow
    glPushMatrix();
    glColor3f(0.0f, 0.0f, 0.0f);
    glTranslatef(0.1f, 0.4f, 1.55f);//right
    glutSolidSphere(0.15f, 100, 100);
    glTranslatef(-0.8f, 0.0f, -0.08f); //left
    glutSolidSphere(0.15f, 100, 100);
    glTranslatef(1.38f, 0.58f, 0.06f); //right up
    glutSolidSphere(0.2f, 100, 100);
    glTranslatef(-1.95f, -0.02f, -0.08f);//left up
    glutSolidSphere(0.2f, 100, 100);
    glPopMatrix();

    GLUquadric* quad = gluNewQuadric();

    //right brow
    glPushMatrix();
    glColor3f(0.0f, 0.0f, 0.0f);
    glTranslatef(0.10f, 0.40f, 1.55f);
    glRotatef(90, 0, 1, 0); //y
    glRotatef(-45, 1, 0, 0);//x
    //glRotatef(20, 0, 0, 1);//z
    gluCylinder(quad, 0.15f, 0.2f, 0.8f, 100, 100);
    glPopMatrix();

    //left brow
    glPushMatrix();
    glColor3f(0.0f, 0.0f, 0.0f);
    glTranslatef(-0.7f, 0.40f, 1.45f);
    glRotatef(-90, 0, 1, 0);
    glRotatef(-45, 1, 0, 0);
    gluCylinder(quad, 0.15f, 0.2f, 0.8f, 100, 100);
    glPopMatrix();
}

void drawMouth()
{
    GLfloat matAmbient[] = { 1.0, 0.25, 0.4, 1.0f };
    GLfloat matDiffuse[] = { 1.0, 0.25, 0.4, 1.0f };
    GLfloat matSpecular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    GLfloat matShininess[] = { 50.0f };

    glMaterialfv(GL_FRONT, GL_AMBIENT, matAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, matDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, matSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, matShininess);

    GLUquadric* quad = gluNewQuadric();

    //mouth
    glPushMatrix();
    glTranslatef(-0.35, -0.7, 1.35);
    glRotatef(90, 0, 1, 0);
    gluCylinder(quad, 0.05, 0.05, 0.4, 100, 100);
    glPopMatrix();

    glPushMatrix();
    glTranslatef(-0.3, -0.7, 1.35);
    glutSolidSphere(0.05, 100, 100);
    glPopMatrix();

    glPushMatrix();
    glTranslatef(0, -0.7, 1.35);
    glutSolidSphere(0.05, 100, 100);
    glPopMatrix();


}

void drawHair()
{
    // Set material properties for the hair
    GLfloat matAmbient[] = { 0.0f, 0.0f, 0.0f, 1.0f };
    GLfloat matDiffuse[] = { 0.0f, 0.0f, 0.0f, 1.0f };
    GLfloat matSpecular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    GLfloat matShininess[] = { 50.0f };

    glMaterialfv(GL_FRONT, GL_AMBIENT, matAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, matDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, matSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, matShininess);

    glPushMatrix();
    glColor3f(0.0f, 0.0f, 0.0f);
    glTranslatef(1.35f, 1.25f, -1.1f);
    glutSolidSphere(1.8f, 100, 100);
    glPopMatrix();

    //eyes
    glPushMatrix();
    glColor3f(0.0f, 0.0f, 0.0f);
    glLineWidth(5.0f);
    glBegin(GL_LINES);
    glVertex3f(1.4f, 1.0f, 1.05f);//right
    glVertex3f(2.0f, 1.0f, 1.05f);
    glEnd();

    glBegin(GL_LINES);
    glVertex3f(0.7f, 1.0f, 0.90f);//left
    glVertex3f(0.1f, 1.0f, 0.90f);
    glEnd();

    glLineWidth(1.0f);
    glBegin(GL_LINES); //right
    glVertex3f(1.4, 1.1f, 1.05f);
    glVertex3f(2, 1.2f, 1.05f);
    glEnd();

    glBegin(GL_LINES); //left
    glVertex3f(0.5, 1.1f, 1.05f);
    glVertex3f(-0.1, 1.2f, 1.05f);
    glEnd();

    glColor3f(0, 0, 0);
    glTranslatef(0.5, 0.9, 0.9);
    drawCircle(0.1, 100);
    glTranslatef(1.0, 0, 0.15);
    drawCircle(0.1, 100);


    glPopMatrix();

}

void drawBody()
{
    // Enable lighting
    glEnable(GL_LIGHTING);

    // Draw the cuboid body


    // Set material properties for the red color
    GLfloat matAmbient[] = { 1.0f, 0.0f, 0.0f, 1.0f };
    GLfloat matDiffuse[] = { 1.0f, 0.0f, 0.0f, 1.0f };
    GLfloat matSpecular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    GLfloat matShininess[] = { 50.0f };
    glMaterialfv(GL_FRONT, GL_AMBIENT, matAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, matDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, matSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, matShininess);

    GLUquadric* quad = gluNewQuadric();
    glPushMatrix();
    glTranslatef(0.05f, -1.0f, -0.5f);
    glRotatef(90, 1, 0, 0);
    gluCylinder(quad, 1.25, 1.25, 1.55, 100, 100);//body
    glPopMatrix();

}

void drawLeftHand() {
    GLfloat matAmbient[] = { 1.0f, 0.0f, 0.0f, 1.0f };
    GLfloat matDiffuse[] = { 1.0f, 0.0f, 0.0f, 1.0f };
    GLfloat matSpecular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    GLfloat matShininess[] = { 50.0f };
    glMaterialfv(GL_FRONT, GL_AMBIENT, matAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, matDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, matSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, matShininess);

    GLUquadric* quad = gluNewQuadric();

    glPushMatrix(); //hand left
    glTranslatef(-0.7, -1.7, 0.34);
    glRotatef(90, 1, 0, 0);
    glRotatef(-21, 1, 0, 0);

    glRotatef(10, 0, 1, 0);
    gluCylinder(quad, 0.17, 0.17, 1.1, 100, 100);
    glPopMatrix();

    GLfloat HandAmbient[] = { 1.0f, 0.831f, 0.718f, 1.0f };
    GLfloat HandDiffuse[] = { 1.0f, 0.831f, 0.718f, 1.0f };
    GLfloat HandSpecular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    GLfloat HandShininess[] = { 50.0f };

    glMaterialfv(GL_FRONT, GL_AMBIENT, HandAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, HandDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, HandSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, HandShininess);

    glPushMatrix();
    glTranslatef(-0.5, -2.7, 0.8);
    glutSolidSphere(0.16, 100, 100);
    glTranslatef(0, -0.1, 0.05);
    glutSolidSphere(0.2, 100, 100);
    glPopMatrix();
}

void drawRightHand() {
    GLfloat matAmbient[] = { 1.0f, 0.0f, 0.0f, 1.0f };
    GLfloat matDiffuse[] = { 1.0f, 0.0f, 0.0f, 1.0f };
    GLfloat matSpecular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    GLfloat matShininess[] = { 50.0f };
    glMaterialfv(GL_FRONT, GL_AMBIENT, matAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, matDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, matSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, matShininess);

    GLUquadric* quad = gluNewQuadric();

    glPushMatrix(); //hand right
    glTranslatef(0.7f, -1.7, 0.4);
    glRotatef(90, 1, 0, 0);
    glRotatef(-20, 1, 0, 0);
    glRotatef(-20, 0, 1, 0);
    gluCylinder(quad, 0.17, 0.17, 1.1, 100, 100);
    glPopMatrix();

    GLfloat HandAmbient[] = { 1.0f, 0.831f, 0.718f, 1.0f };
    GLfloat HandDiffuse[] = { 1.0f, 0.831f, 0.718f, 1.0f };
    GLfloat HandSpecular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    GLfloat HandShininess[] = { 50.0f };

    glMaterialfv(GL_FRONT, GL_AMBIENT, HandAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, HandDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, HandSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, HandShininess);

    glPushMatrix();
    glTranslatef(0.32, -2.7, 0.8);
    glutSolidSphere(0.16, 100, 100);
    glTranslatef(0, -0.1, 0.05);
    glutSolidSphere(0.2, 100, 100);
    glPopMatrix();

}

void drawPants()
{
    GLfloat matAmbient[] = { 1.0f, 1.0f, 0.0f, 1.0f };
    GLfloat matDiffuse[] = { 1.0f, 1.0f, 0.0f, 1.0f };
    GLfloat matSpecular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    GLfloat matShininess[] = { 50.0f };

    glMaterialfv(GL_FRONT, GL_AMBIENT, matAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, matDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, matSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, matShininess);

    glPushMatrix();
    glTranslatef(0.05, -2.29, -0.5);
    glutSolidSphere(1.23, 100, 100);
    glPopMatrix();
}

void drawleg()
{


    GLfloat matAmbient[] = { 1.0f, 0.831f, 0.718f, 1.0f };
    GLfloat matDiffuse[] = { 1.0f, 0.831f, 0.718f, 1.0f };
    GLfloat matSpecular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    GLfloat matShininess[] = { 50.0f };

    glMaterialfv(GL_FRONT, GL_AMBIENT, matAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, matDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, matSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, matShininess);

    glPushMatrix();
    glTranslatef(0.6, -3.2, 0.75);
    glutSolidSphere(0.3, 100, 100);
    glPopMatrix();

    glPushMatrix();
    glTranslatef(-0.64, -3.2, 0.75);
    glutSolidSphere(0.3, 100, 100);
    glPopMatrix();

    GLfloat legAmbient[] = { 1.0f, 1.0f, 0.0f, 1.0f };
    GLfloat legDiffuse[] = { 1.0f, 1.0f, 0.0f, 1.0f };
    GLfloat legSpecular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    GLfloat legShininess[] = { 50.0f };

    glMaterialfv(GL_FRONT, GL_AMBIENT, legAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, legDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, legSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, legShininess);

    GLUquadric* quad = gluNewQuadric();

    glPushMatrix();
    glTranslatef(-0.27, -2.74, -0.5);
    glRotatef(17, 1, 0, 0);
    glRotatef(-15, 0, 1, 0);
    gluCylinder(quad, 0.68, 0.28, 1.5, 100, 100);
    glPopMatrix();

    glPushMatrix();
    glTranslatef(0.27, -2.74, -0.5);
    glRotatef(17, 1, 0, 0);
    glRotatef(13, 0, 1, 0);
    gluCylinder(quad, 0.68, 0.28, 1.5, 100, 100);
    glPopMatrix();



}

void drawCushion()
{
    GLfloat matAmbient[] = { 0.1843f, 0.9765f, 0.1412f, 1.0f };
    GLfloat matDiffuse[] = { 0.1843f, 0.9765f, 0.1412f, 1.0f };
    GLfloat matSpecular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    GLfloat matShininess[] = { 50.0f };

    glMaterialfv(GL_FRONT, GL_AMBIENT, matAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, matDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, matSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, matShininess);

    glPushMatrix();
    glColor3f(0.0f, 1.0f, 0.0f);
    glTranslatef(0, -3.5, -0.4);
    glScalef(2, 0.3, 2);
    glutSolidCube(1.3);
    glPopMatrix();

    GLUquadric* quad = gluNewQuadric();

    glPushMatrix();
    glTranslatef(-1.3, -3.5, 0.87);
    glRotatef(90, 0, 1, 0);
    gluCylinder(quad, 0.2, 0.2, 2.6, 100, 100); //FRONT
    glPopMatrix();

    glPushMatrix();
    glTranslatef(-1.3, -3.5, -1.75);
    glRotatef(90, 0, 1, 0);
    gluCylinder(quad, 0.2, 0.2, 2.6, 100, 100); //BACK
    glPopMatrix();

    glPushMatrix();
    glTranslatef(-1.3, -3.5, 0.87);
    glRotatef(-180, 0, 1, 0);
    gluCylinder(quad, 0.2, 0.2, 2.6, 100, 100); //LEFT
    glPopMatrix();

    glPushMatrix();
    glTranslatef(1.3, -3.5, 0.87);
    glRotatef(180, 0, 1, 0);
    gluCylinder(quad, 0.2, 0.2, 2.6, 100, 100); //RIGHT
    glPopMatrix();

    glPushMatrix();
    glTranslatef(1.3, -3.5, 0.87);
    glutSolidSphere(0.2, 100, 100); //RIGHT
    glPopMatrix();

    glPushMatrix();
    glTranslatef(-1.3, -3.5, 0.87);
    glutSolidSphere(0.2, 100, 100); //RIGHT
    glPopMatrix();

    glPushMatrix();
    glTranslatef(-1.3, -3.5, -1.75);
    glutSolidSphere(0.2, 100, 100); //RIGHT BACK
    glPopMatrix();

    glPushMatrix();
    glTranslatef(1.3, -3.5, -1.75);
    glutSolidSphere(0.2, 100, 100); //LEFT BACK
    glPopMatrix();
}

void shakeHead()
{
    if (shakingHead) {
        glRotatef(shakeAngle, 0.0f, 1.0f, 0.0f);
    }
    drawHair();
    drawFace();
    drawMouth();
}

void Operation() {

    drawCushion();

    glPushMatrix();
    glTranslatef(-1.35, -0.9, 0.46);
    if (LieDown || LieDownComplete) {
        glTranslatef(0, 0, TranslatePosition);
        glRotatef(rotationAngle, 1.0, 0.0, 0.0); // Rotate around x-axis
    }

    shakeHead(); //================shake/dance=====================
    drawBody();
    drawPants();
    glPopMatrix();

    //========RIGHT HAND============
    glPushMatrix();
    if (LieDown3 || LieDownComplete3) {
        glTranslatef(0, 0, TranslatePosition4);
        glRotatef(rotationAngle3, 1, 0, 0);
        glRotatef(rotationAngle4, 0, 1, 0);
        glRotatef(rotationAngle6, 1, 0, 0);
        glTranslatef(TranslatePosition5, 0, 0);
    }
    drawRightHand();
    glFlush();
    glPopMatrix();

    //=========LEG================
    glPushMatrix();
    if (LieDown1 || LieDownComplete1) {
        glTranslatef(0, TranslatePosition1, 0);
    }
    drawleg();
    glPopMatrix();

    //==============LEFT HAND===============
    glPushMatrix();
    if (LieDown2 || LieDownComplete2) {
        glTranslatef(TranslatePosition2, 0, 0); //x
        glRotatef(rotationAngle2, 0, 1, 0);
        glRotatef(rotationAngle5, 1, 0, 0);
        glTranslatef(0, 0, TranslatePosition6);//y
        glTranslatef(TranslatePosition7, 0, 0); //z
    }
    drawLeftHand();
    glFlush();
    glPopMatrix();

   

}

struct Shape {
    enum Type { Triangle, Circle, Square };

    Type type;
    float x;
    float y;
    float size;
};

const int numTriangles = 40; // Number of random triangles
const int numCircles = 35;   // Number of random circles
const int numSquares = 40;   // Number of random squares
const float maxShapeSize = 2.0f; // Maximum size of each shape
const float maxTranslation = 100.0f; // Maximum translation range

std::vector<Shape> shapes;

bool checkOverlap(float x, float y, float size, Shape::Type type) {
    // Check if the new shape overlaps with any existing shapes
    for (const auto& shape : shapes) {
        if (shape.type == type) {
            float dx = x - shape.x;
            float dy = y - shape.y;
            float distance = sqrt(dx * dx + dy * dy);

            // Shapes overlap if distance between their centers is less than half the sum of their sizes
            if (distance < (size + shape.size) / 2.0f) {
                return true; // Overlap detected
            }
        }
    }

    return false; // No overlap
}

void initShapes() {
    srand(time(NULL));

    for (int i = 0; i < numTriangles; ++i) {
        float x, y, size;

        // Generate random positions and size for triangle, checking for overlap
        do {
            x = static_cast<float>(rand()) / RAND_MAX * maxTranslation - maxTranslation / 2.0f;
            y = static_cast<float>(rand()) / RAND_MAX * maxTranslation - maxTranslation / 2.0f;
            size = static_cast<float>(rand()) / RAND_MAX * maxShapeSize + 1.0f;
        } while (checkOverlap(x, y, size, Shape::Triangle) || checkOverlap(x, y, size, Shape::Square) || checkOverlap(x, y, size, Shape::Circle));

        Shape triangle = { Shape::Triangle, x, y, size };
        shapes.push_back(triangle);
    }

    for (int i = 0; i < numCircles; ++i) {
        float x, y, size;

        // Generate random positions and size for circle, checking for overlap with triangles and other circles
        do {
            x = static_cast<float>(rand()) / RAND_MAX * maxTranslation - maxTranslation / 2.0f;
            y = static_cast<float>(rand()) / RAND_MAX * maxTranslation - maxTranslation / 2.0f;
            size = static_cast<float>(rand()) / RAND_MAX * maxShapeSize + 1.0f;
        } while (checkOverlap(x, y, size, Shape::Circle) || checkOverlap(x, y, size, Shape::Triangle) || checkOverlap(x, y, size, Shape::Square));

        Shape circle = { Shape::Circle, x, y, size };
        shapes.push_back(circle);
    }

    for (int i = 0; i < numSquares; ++i) {
        float x, y, size;

        // Generate random positions and size for square, checking for overlap with triangles, circles, and other squares
        do {
            x = static_cast<float>(rand()) / RAND_MAX * maxTranslation - maxTranslation / 2.0f;
            y = static_cast<float>(rand()) / RAND_MAX * maxTranslation - maxTranslation / 2.0f;
            size = static_cast<float>(rand()) / RAND_MAX * maxShapeSize + 1.0f;
        } while (checkOverlap(x, y, size, Shape::Square) || checkOverlap(x, y, size, Shape::Triangle) || checkOverlap(x, y, size, Shape::Circle));

        Shape square = { Shape::Square, x, y, size };
        shapes.push_back(square);
    }
}

//background
void drawShapes() {
    glPushMatrix();
    glTranslatef(0.0f, 0.0f, -30.0f); // Move the shapes further away from the camera

    // Disable lighting before drawing the shapes
    glDisable(GL_LIGHTING);

    // Draw each shape
    glBegin(GL_TRIANGLES);
    glColor3f(1.0f, 0.5f, 0.5f); // Light red color for triangles

    for (const auto& shape : shapes) {
        if (shape.type == Shape::Triangle) {
            float x = shape.x;
            float y = shape.y;
            float size = shape.size;

            glVertex3f(x, y, 0.0f);
            glVertex3f(x + size, y, 0.0f);
            glVertex3f(x + size / 2.0f, y + size * sqrt(3.0f) / 2.0f, 0.0f);
        }
    }

    glEnd();


    glColor3f(0.992f, 0.914f, 0.573f); // Light blue color for circles

    for (const auto& shape : shapes) {
        if (shape.type == Shape::Circle) {
            float x = shape.x;
            float y = shape.y;
            float size = shape.size;

            // Draw circle using many line segments to approximate a circle
            const int numSegments = 50;
            glBegin(GL_POLYGON);
            for (int i = 0; i < numSegments; ++i) {
                float theta = 2.0f * M_PI * float(i) / float(numSegments);  // Get the current angle
                float dx = size * cosf(theta);  // Calculate the x component
                float dy = size * sinf(theta);  // Calculate the y component
                glVertex2f(x + dx, y + dy);  // Specify the vertex
            }
            glEnd();
        }
    }



    glBegin(GL_QUADS);
    glColor3f(0.5f, 1.0f, 0.5f); // Light green color for squares

    for (const auto& shape : shapes) {
        if (shape.type == Shape::Square) {
            float x = shape.x;
            float y = shape.y;
            float size = shape.size;

            glVertex3f(x, y, 0.0f);
            glVertex3f(x + size, y, 0.0f);
            glVertex3f(x + size, y + size, 0.0f);
            glVertex3f(x, y + size, 0.0f);
        }
    }

    glEnd();

    glPopMatrix();
    glEnable(GL_LIGHTING);
}



void initLighting()
{
    //enable the lighting
    glEnable(GL_LIGHTING);

    //Enable light source 0
    glEnable(GL_LIGHT0);

    GLfloat lightAmbient[] = { 0.2f, 0.2f,0.2f,1.0f };
    GLfloat lightDiffuse[] = { 0.8f, 0.8f, 0.8f, 1.0f };
    GLfloat lightSpecular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    GLfloat lightPosition[] = { 0.0f, 5.0f, 5.0f, 1.0f };

    glLightfv(GL_LIGHT0, GL_AMBIENT, lightAmbient);
    glLightfv(GL_LIGHT0, GL_DIFFUSE, lightDiffuse);
    glLightfv(GL_LIGHT0, GL_SPECULAR, lightSpecular);
    glLightfv(GL_LIGHT0, GL_POSITION, lightPosition);
}

void setMaterialProperties()
{
    GLfloat matAmbient[] = { 1.0f, 0.831f, 0.718f, 1.0f };
    GLfloat matDiffuse[] = { 1.0f, 0.831f, 0.718f, 1.0f };
    GLfloat matSpecular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    GLfloat matShininess[] = { 50.0f };

    glMaterialfv(GL_FRONT, GL_AMBIENT, matAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, matDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, matSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, matShininess);
}

void renderBitmap(float x, float y, void* font, char* string)
{
    const char* c;

    glRasterPos2f(x, y);
    for (c = string; *c != '\0'; c++)
    {
        glutBitmapCharacter(font, *c);
    }

}

void displayTexts()
{
    char buf[100] = { 0 };

    glMatrixMode(GL_PROJECTION);
    glPushMatrix();
    glLoadIdentity();
    gluOrtho2D(0, 800, 0, 600);

    glMatrixMode(GL_MODELVIEW);
    glPushMatrix();
    glLoadIdentity();

    glDisable(GL_DEPTH_TEST);
    glDisable(GL_LIGHTING);

    glColor3f(0.0f, 0.0f, 0.0f);
    sprintf_s(buf, "PRESS 'L / l' TO ROTATE LEFT");
    renderBitmap(10, 30, GLUT_BITMAP_HELVETICA_10, buf);
    sprintf_s(buf, "PRESS 'R / r' TO ROTATE RIGHT");
    renderBitmap(10, 60, GLUT_BITMAP_HELVETICA_10, buf);
    sprintf_s(buf, "PRESS 'B / b' TO LIE DOWN");
    renderBitmap(10, 100, GLUT_BITMAP_HELVETICA_10, buf);
    sprintf_s(buf, "PRESS 'Z / z' TO RESET");
    renderBitmap(10, 130, GLUT_BITMAP_HELVETICA_10, buf);
    sprintf_s(buf, "PRESS 'S / s' TO SHAKE/DANCE");
    renderBitmap(10, 160, GLUT_BITMAP_HELVETICA_10, buf);
    sprintf_s(buf, "PRESS 'P / p' TO PLAY MUSIC");
    renderBitmap(10, 190, GLUT_BITMAP_HELVETICA_10, buf);
    sprintf_s(buf, "PRESS 'O / o' TO STOP THE MUSIC");
    renderBitmap(10, 220, GLUT_BITMAP_HELVETICA_10, buf);

    glMatrixMode(GL_PROJECTION);
    glPopMatrix();
    glMatrixMode(GL_MODELVIEW);
    glPopMatrix();

    glEnable(GL_DEPTH_TEST);
    glEnable(GL_LIGHTING);
}

void idle() {
    // Update the scale factor to create the heartbeat effect
    if (increasing) {
        scale += delta;
        if (scale >= 1.2f) { // Upper limit for scaling
            increasing = false;
        }
    }
    else {
        scale -= delta;
        if (scale <= 1.0f) { // Lower limit for scaling
            increasing = true;
        }
    }

    if (shakingHead)
    {
        shakeAngle = shakeSpeed * sin(shakeTime * 30.0f); // Shake head
        shakeTime += 0.01f; // Increment time

        if (shakeTime >= shakeDuration) // Stop shaking after 5 seconds
        {
            shakingHead = false;
            shakeAngle = 0.0f;
        }
    }

    if (LieDown && !LieDownComplete) {
        rotationAngle -= LieDownSpeed;
        TranslatePosition -= TranslateSpeed;
        if (rotationAngle <= targetAngle && TranslatePosition <= TranslatePosition) {
            rotationAngle = targetAngle;
            TranslatePosition = targetPosition;
            LieDown = false;
            LieDownComplete = true;
        }

    }

    if (LieDown1 && !LieDownComplete1) {
        TranslatePosition1 += TranslateSpeed1;
        if (TranslatePosition1 >= targetPosition1) {
            TranslatePosition1 = targetPosition1;
            LieDown1 = false;
            LieDownComplete1 = true;
        }

    }

    if (LieDown2 && !LieDownComplete2) {
        rotationAngle2 -= LieDownSpeed2;
        TranslatePosition2 -= TranslateSpeed2;
        TranslatePosition6 -= TranslateSpeed6;
        TranslatePosition7 -= TranslateSpeed7;
        rotationAngle5 -= LieDownSpeed5;
        if (rotationAngle2 <= targetAngle2 && TranslatePosition2 <= targetPosition2 && rotationAngle5 <= targetAngle5 && TranslatePosition6 >= targetPosition6 && TranslatePosition7 >= targetPosition7 && rotationAngle6 >= targetAngle6) {
            rotationAngle2 = targetAngle2;
            TranslatePosition2 = targetPosition2;
            rotationAngle5 = targetAngle5;
            TranslatePosition6 = targetPosition6;
            TranslatePosition7 = targetPosition7;
            LieDown2 = false;
            LieDownComplete2 = true;
        }

    }

    if (LieDown3 && !LieDownComplete3) {
        rotationAngle3 -= LieDownSpeed3;
        TranslatePosition4 -= TranslateSpeed4;
        rotationAngle4 += LieDownSpeed4;
        rotationAngle6 -= LieDownSpeed6;
        TranslatePosition5 += TranslateSpeed5;
        if (rotationAngle3 <= targetAngle3 && TranslatePosition4 >= targetPosition4 && rotationAngle4 >= targetAngle4 && TranslatePosition5 <= targetPosition5 && rotationAngle6 <= targetAngle6) {
            rotationAngle3 = targetAngle3;
            TranslatePosition4 = targetPosition4;
            rotationAngle4 = targetAngle4;
            TranslatePosition5 = targetPosition5;
            rotationAngle6 = targetAngle6;
            LieDown3 = false;
            LieDownComplete3 = true;
        }
    }

    // Request a redisplay to update the frame
    glutPostRedisplay();
}

void drawFloor() {
    glDisable(GL_LIGHTING);
    //glColor3f(0.5961f, 0.9843f, 0.5961f);
    glColor3f(0.9624f, 0.7073f, 0.3624f);
    glBegin(GL_QUADS);
    glVertex3f(-25.0f, -4.0f, 11.0f);
    glVertex3f(25.0f, -4.0f, 11.0f);
    glVertex3f(25.0f, -4.0f, -25.0f);
    glVertex3f(-25.0f, -4.0f, -25.0f);
    glEnd();
    glEnable(GL_LIGHTING);
}

void drawTable() {
    // Set material properties for the table
    GLfloat matAmbient[] = { 0.5f, 0.35f, 0.05f, 1.0f };
    GLfloat matDiffuse[] = { 0.5f, 0.35f, 0.05f, 1.0f };
    GLfloat matSpecular[] = { 0.2f, 0.2f, 0.2f, 1.0f };
    GLfloat matShininess[] = { 10.0f };

    glMaterialfv(GL_FRONT, GL_AMBIENT, matAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, matDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, matSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, matShininess);

    // Draw the table top
    glPushMatrix();
    glTranslatef(0.0f, -2.2f, -5.5f); // Positioning the table relative to the character
    glScalef(10.0f, 0.5f, 1.0f); // Scale to make a flat table top
    glutSolidCube(1.0f);
    glPopMatrix();

    // Draw the table legs
    glPushMatrix();
    glTranslatef(-4.5f, -2.5f, -5.5f); // Leg front left
    glScalef(0.5f, 1.0f, 0.1f);
    glutSolidCube(1.0f);
    glPopMatrix();

    glPushMatrix();
    glTranslatef(4.5f, -2.5f, -5.5f); // Leg front right
    glScalef(0.5f, 1.0f, 0.1f);
    glutSolidCube(1.0f);
    glPopMatrix();

    glPushMatrix();
    glTranslatef(-4.5f, -2.5f, -6.5f); // Leg back left
    glScalef(0.5f, 1.0f, 0.1f);
    glutSolidCube(1.0f);
    glPopMatrix();

    glPushMatrix();
    glTranslatef(4.5f, -2.5f, -6.5f); // Leg back right
    glScalef(0.5f, 1.0f, 0.1f);
    glutSolidCube(1.0f);
    glPopMatrix();
}

void drawTeapot() {
    // Set material properties for the teapot
    GLfloat matAmbient[] = { 0.8667f, 0.6471f, 0.1255f, 1.0f };
    GLfloat matDiffuse[] = { 0.8667f, 0.6471f, 0.1255f, 1.0f };
    GLfloat matSpecular[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    GLfloat matShininess[] = { 50.0f };

    glMaterialfv(GL_FRONT, GL_AMBIENT, matAmbient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, matDiffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, matSpecular);
    glMaterialfv(GL_FRONT, GL_SHININESS, matShininess);

    // Position and draw the teapot
    glPushMatrix();
    glTranslatef(3.0f, -1.5f, -5.0f); 
    glScalef(1.0f, 1.0f, 1.0f); 
    glutSolidTeapot(0.5f);
    glPopMatrix();
}



void display() {
    if (deltaMove || deltaStrafe)
        computePos(deltaMove, deltaStrafe);

    //Clear color and depth buffers
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    glLoadIdentity();

    //Reset transformations
    glLoadIdentity();
    //Set the camera
    gluLookAt(10, 0, 15,  // Eye position
        x + lx, 1.0f, z + lz,    // Look at point
        0.0, 1.0, 0.0);

    setMaterialProperties();
    drawShapes();
    
    drawFloor();

    drawTable();
    drawTeapot();

    glRotatef(angle, 0, 1, 0);
    Operation();
    displayTexts();
    glutSwapBuffers();
    glFlush();
}




void keyboard(unsigned char key, int xx, int yy)
{
    switch (key) {
    case 'L':
    case 'l':
        angle -= 5;  // Rotate left
        break;
    case 'R':
    case 'r':
        angle += 5;  // Rotate right
        break;
    case 'b':
    case 'B':
        if (!LieDown && !LieDownComplete) {
            LieDown = true; // Start the animation
        }

        if (!LieDown1 && !LieDownComplete1) {
            LieDown1 = true; // Start the animation
        }

        if (!LieDown2 && !LieDownComplete2) {
            LieDown2 = true; // Start the animation
        }

        if (!LieDown3 && !LieDownComplete3) {
            LieDown3 = true; // Start the animation
        }

        //Operation();
        break;
    case 'p':
    case 'P':
        printf("playing the sound...");
        PlaySound(TEXT("C:\\Nitizyou.wav"), NULL, SND_ASYNC);
        break;
    case 'o':
    case 'O':
        printf("Stopping the song...");
        PlaySound(NULL, 0, 0);
        break;
    case'z':
    case'Z':// Reset key
        angle = 0;
       // lx = 0.0;
       // lz = -1.0;
        x = 0;
        z = 5;
        rotationAngle = 0;
        rotationAngle2 = 0;
        rotationAngle3 = 0;
        rotationAngle4 = 0;
        rotationAngle5 = 0;
        rotationAngle6 = 0;
        TranslatePosition = 0;
        TranslatePosition1 = 0;
        TranslatePosition2 = 0;
        TranslatePosition4 = 0;
        TranslatePosition5 = 0;
        TranslatePosition6 = 0;
        TranslatePosition7 = 0;
        //scale = initialScale;

        // Reset any flags
        LieDown = false;
        LieDownComplete = false;
        LieDown1 = false;
        LieDownComplete1 = false;
        LieDown2 = false;
        LieDownComplete2 = false;
        LieDown3 = false;
        LieDownComplete3 = false;
        break;
    case 's':
    case 'S':
        PlaySound(TEXT("C:\\ohooo.wav"), NULL, SND_ASYNC);
        shakingHead = true; // Start head shaking
        shakeTime = 0.0f;
        break;

    case 27:  // Escape key
        exit(0);
        break;
    default:
        break;
    }
    glFlush();
    glutPostRedisplay();
}

void mouseMove(int x, int y)
{
    //this will only be true when the left button is down
    if (xOrigin >= 0)
    {
        //update deltaAngle
        deltaAngle = (x - xOrigin) * 0.001f;

        //update camera's direction
        lx = sin(angle + deltaAngle);
        lz = -cos(angle + deltaAngle);
    }
}

void mouseButton(int button, int state, int x, int y)
{
    if (button == GLUT_LEFT_BUTTON)
    {
        //when the button is released
        if (state == GLUT_UP)
        {
            angle += deltaAngle;
            xOrigin = -1;
        }
        else //state = GLUT_DOWN
        {
            xOrigin = x;
        }
    }
}

int main(int argc, char** argv)
{
    //initialize GLUT and create window
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DEPTH | GLUT_DOUBLE | GLUT_RGBA);
    glutInitWindowPosition(200, 100);
    glutInitWindowSize(800, 600);
    glutCreateWindow("Shin Chan 3D Model");

    //register callbacks
    glutDisplayFunc(display);
    glutReshapeFunc(changeSize);
    glutKeyboardFunc(keyboard);
    
    //OpenGL init
    glEnable(GL_DEPTH_TEST);
    glClearColor(0.8f, 1.0f, 0.8f, 1.0f); // Set the background color to LIGHT GREEN
    initShapes();

    glutMouseFunc(mouseButton);
    glutMotionFunc(mouseMove);

    initLighting();
    glutIdleFunc(idle);
    glutMainLoop();

    return 0;
}