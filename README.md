# fantastic-robot
Say Hi Robot

package com.brentaureli.mariobros.Sprites;
import com.badlogic.gdx.audio.Music;
import com.badlogic.gdx.audio.Sound;
import com.badlogic.gdx.graphics.g2d.Animation;
import com.badlogic.gdx.graphics.g2d.Batch;
import com.badlogic.gdx.graphics.g2d.Sprite;
import com.badlogic.gdx.graphics.g2d.TextureRegion;
import com.badlogic.gdx.math.Vector2;
import com.badlogic.gdx.physics.box2d.Body;
import com.badlogic.gdx.physics.box2d.BodyDef;
import com.badlogic.gdx.physics.box2d.CircleShape;
import com.badlogic.gdx.physics.box2d.EdgeShape;
import com.badlogic.gdx.physics.box2d.Filter;
import com.badlogic.gdx.physics.box2d.Fixture;
import com.badlogic.gdx.physics.box2d.FixtureDef;
import com.badlogic.gdx.physics.box2d.World;
import com.badlogic.gdx.utils.Array;
import com.brentaureli.mariobros.MarioBros;
import com.brentaureli.mariobros.Screens.PlayScreen;
import com.brentaureli.mariobros.Sprites.Other.FireBall;
import com.brentaureli.mariobros.Sprites.Enemies.*;
/**
 * Created by brentaureli on 8/27/15.
 */
public class Mario extends Sprite {
 dddpower Modified 12/26/2016, 12:03:50 AM

part 8
    public enum State { FALLING, JUMPING, STANDING, RUNNING, GROWING, DEAD };
    public State currentState;
 pumpy7 Modified 10/28/2016, 2:50:36 PM
Declared in tutorial 11 to create jump and run animations.
    public State previousState;
 pumpy7 Modified 10/28/2016, 2:51:43 PM
Just like currentState from the above line, refer to tutorial11
    public World world;
    public Body b2body;
    private TextureRegion marioStand;
 dddpower Modified 12/26/2016, 12:41:59 AM

    private Animation marioRun;
 pumpy7 Modified 10/28/2016, 2:52:42 PM
from badlogic.gdx
    private TextureRegion marioJump;
    private TextureRegion marioDead;
    private TextureRegion bigMarioStand;
 dddpower Modified 12/28/2016, 1:06:12 AM

    private TextureRegion bigMarioJump;
    private Animation bigMarioRun;
    private Animation growMario;
    private float stateTimer;
    private boolean runningRight;
 pumpy7 Modified 10/28/2016, 2:54:04 PM
Boolean value indicating the direction of Mario's move.
If True Mario is moving to the right. IF False Mario is moving to the left
    private boolean marioIsBig;
    private boolean runGrowAnimation;
    private boolean timeToDefineBigMario;
    private boolean timeToRedefineMario;
    private boolean marioIsDead;
    private PlayScreen screen;
    private Array<FireBall> fireballs;
    public Mario(PlayScreen screen){
//initialize default values
        this.screen = screen;
        this.world = screen.getWorld();
        currentState = State.STANDING;
 pumpy7 Modified 10/28/2016, 2:55:03 PM
Initializing values
        previousState = State.STANDING;
 pumpy7 Modified 10/28/2016, 2:55:14 PM
        stateTimer = 0;
 pumpy7 Modified 10/28/2016, 2:55:17 PM
        runningRight = true;
 pumpy7 Modified 10/28/2016, 2:55:21 PM
        Array<TextureRegion> frames = new Array<TextureRegion>();
 pumpy7 Modified 10/28/2016, 2:56:20 PM
Create an array of texture regions to pass the constructor for the animation
//get run animation frames and add them to marioRun Animation
        for(int i = 1; i < 4; i++)
 pumpy7 Modified 10/28/2016, 3:16:00 PM
i starts iterates from 1 to 3 because the animation 1 to 3 is for running Mario animation.
Remember array indexing starts from 0
graphic assets can be found at android / asset / mario and enemies.png
            frames.add(new TextureRegion(screen.getAtlas().findRegion("little_mario"), i * 16, 0, 16, 16));
 pumpy7 Modified 10/28/2016, 3:17:39 PM
16 denotes for 16 pixels here
basically specifying 16 by 16 pixels regions from the graphic asset
        marioRun = new Animation(0.1f, frames);
        frames.clear();
 dddpower Modified 12/28/2016, 1:07:33 AM

        for(int i = 1; i < 4; i++)
            frames.add(new TextureRegion(screen.getAtlas().findRegion("big_mario"), i * 16, 0, 16, 32));
        bigMarioRun = new Animation(0.1f, frames);
        frames.clear();
//get set animation frames from growing mario
        frames.add(new TextureRegion(screen.getAtlas().findRegion("big_mario"), 240, 0, 16, 32));
        frames.add(new TextureRegion(screen.getAtlas().findRegion("big_mario"), 0, 0, 16, 32));
        frames.add(new TextureRegion(screen.getAtlas().findRegion("big_mario"), 240, 0, 16, 32));
        frames.add(new TextureRegion(screen.getAtlas().findRegion("big_mario"), 0, 0, 16, 32));
        growMario = new Animation(0.2f, frames);
//get jump animation frames and add them to marioJump Animation
        marioJump = new TextureRegion(screen.getAtlas().findRegion("little_mario"), 80, 0, 16, 16);
        bigMarioJump = new TextureRegion(screen.getAtlas().findRegion("big_mario"), 80, 0, 16, 32);
//create texture region for mario standing
        marioStand = new TextureRegion(screen.getAtlas().findRegion("little_mario"), 0, 0, 16, 16);
        bigMarioStand = new TextureRegion(screen.getAtlas().findRegion("big_mario"), 0, 0, 16, 32);
//create dead mario texture region
        marioDead = new TextureRegion(screen.getAtlas().findRegion("little_mario"), 96, 0, 16, 16);
//define mario in Box2d
        defineMario();
//set initial values for marios location, width and height. And initial frame as marioStand.
        setBounds(0, 0, 16 / MarioBros.PPM, 16 / MarioBros.PPM);
        setRegion(marioStand);
        fireballs = new Array<FireBall>();
    }
    public void update(float dt){
 dddpower Modified 12/26/2016, 12:43:54 AM

update
// time is up : too late mario dies T_T
// the !isDead() method is used to prevent multiple invocation
// of "die music" and jumping
// there is probably better ways to do that but it works for now.
        if (screen.getHud().isTimeUp() && !isDead()) {
            die();
        }
//update our sprite to correspond with the position of our Box2D body
 dddpower Modified 12/28/2016, 1:18:34 AM

        if(marioIsBig)
            setPosition(b2body.getPosition().x - getWidth() / 2, b2body.getPosition().y - getHeight() / 2 - 6 / MarioBros.PPM);
        else
            setPosition(b2body.getPosition().x - getWidth() / 2, b2body.getPosition().y - getHeight() / 2);
//update sprite with the correct frame depending on marios current action
        setRegion(getFrame(dt));
        if(timeToDefineBigMario)
            defineBigMario();
        if(timeToRedefineMario)
            redefineMario();
        for(FireBall  ball : fireballs) {
            ball.update(dt);
            if(ball.isDestroyed())
                fireballs.removeValue(ball, true);
        }
    }
    public TextureRegion getFrame(float dt){
 dddpower Modified 12/26/2016, 12:48:23 AM

animation
//get marios current state. ie. jumping, running, standing...
        currentState = getState();
        TextureRegion region;
//depending on the state, get corresponding animation keyFrame.
        switch(currentState){
 dddpower Modified 12/28/2016, 1:11:57 AM

            case DEAD:
                region = marioDead;
                break;
            case GROWING:
                region = growMario.getKeyFrame(stateTimer);
                if(growMario.isAnimationFinished(stateTimer)) {
                    runGrowAnimation = false;
                }
                break;
            case JUMPING:
                region = marioIsBig ? bigMarioJump : marioJump;
                break;
            case RUNNING:
                region = marioIsBig ? bigMarioRun.getKeyFrame(stateTimer, true) : marioRun.getKeyFrame(stateTimer, true);
                break;
            case FALLING:
            case STANDING:
            default:
                region = marioIsBig ? bigMarioStand : marioStand;
                break;
        }
//if mario is running left and the texture isnt facing left... flip it.
        if((b2body.getLinearVelocity().x < 0 || !runningRight) && !region.isFlipX()){
            region.flip(true, false);
            runningRight = false;
        }
//if mario is running right and the texture isnt facing right... flip it.
        else if((b2body.getLinearVelocity().x > 0 || runningRight) && region.isFlipX()){
            region.flip(true, false);
            runningRight = true;
        }
//if the current state is the same as the previous state increase the state timer.
//otherwise the state has changed and we need to reset timer.
        stateTimer = currentState == previousState ? stateTimer + dt : 0;
//update previous state
        previousState = currentState;
//return our final adjusted frame
        return region;
    }
    public State getState(){
 dddpower Modified 12/26/2016, 12:47:30 AM

for animation setting
//Test to Box2D for velocity on the X and Y-Axis
//if mario is going positive in Y-Axis he is jumping... or if he just jumped and is falling remain in jump state
        if(marioIsDead)
            return State.DEAD;
        else if(runGrowAnimation)
            return State.GROWING;
        else if((b2body.getLinearVelocity().y > 0 && currentState == State.JUMPING) || (b2body.getLinearVelocity().y < 0 && previousState == State.JUMPING))
 pumpy7 Modified 10/28/2016, 3:25:25 PM
If b2body's linear velocity in Y direction is greater than 0, meaning mario is going up and current stat is jumping, it must be jumping
if the linear velocity in Y direction is smaller than 0 and previous state was jumping, Mario is still in process of jumping. It just is returning to the ground or falling down from previous jump.
By specifying the falling that is part of jumping animation, we are able to display different animation for jumping and falling.
For jumping motion Mario has his hands up, for falling animation he is just standing.
            return State.JUMPING;
//if negative in Y-Axis mario is falling
        else if(b2body.getLinearVelocity().y < 0)
            return State.FALLING;
//if mario is positive or negative in the X axis he is running
        else if(b2body.getLinearVelocity().x != 0)
            return State.RUNNING;
//if none of these return then he must be standing
        else
            return State.STANDING;
    }
    public void grow(){
 dddpower Modified 12/28/2016, 1:17:20 AM

        if( !isBig() ) {
            runGrowAnimation = true;
            marioIsBig = true;
            timeToDefineBigMario = true;
            setBounds(getX(), getY(), getWidth(), getHeight() * 2);
            MarioBros.manager.get("audio/sounds/powerup.wav", Sound.class).play();
        }
    }
    public void die() {
        if (!isDead()) {
            MarioBros.manager.get("audio/music/mario_music.ogg", Music.class).stop();
            MarioBros.manager.get("audio/sounds/mariodie.wav", Sound.class).play();
            marioIsDead = true;
            Filter filter = new Filter();
            filter.maskBits = MarioBros.NOTHING_BIT;
            for (Fixture fixture : b2body.getFixtureList()) {
                fixture.setFilterData(filter);
            }
            b2body.applyLinearImpulse(new Vector2(0, 4f), b2body.getWorldCenter(), true);
        }
    }
    public boolean isDead(){
 dddpower Modified 12/28/2016, 1:43:10 AM

        return marioIsDead;
    }
    public float getStateTimer(){
        return stateTimer;
    }
    public boolean isBig(){
        return marioIsBig;
    }
    public void jump(){
        if ( currentState != State.JUMPING ) {
            b2body.applyLinearImpulse(new Vector2(0, 4f), b2body.getWorldCenter(), true);
            currentState = State.JUMPING;
        }
    }
    public void hit(Enemy enemy){
 dddpower Modified 12/28/2016, 1:22:12 AM

        if(enemy instanceof Turtle && ((Turtle) enemy).currentState == Turtle.State.STANDING_SHELL)
 dddpower Modified 12/28/2016, 1:36:55 AM

            ((Turtle) enemy).kick(enemy.b2body.getPosition().x > b2body.getPosition().x ? Turtle.KICK_RIGHT : Turtle.KICK_LEFT);
 dddpower Modified 12/28/2016, 1:37:25 AM

        else {
            if (marioIsBig) {
                marioIsBig = false;
                timeToRedefineMario = true;
                setBounds(getX(), getY(), getWidth(), getHeight() / 2);
                MarioBros.manager.get("audio/sounds/powerdown.wav", Sound.class).play();
            } else {
                die();
            }
        }
    }
    public void redefineMario(){
 dddpower Modified 12/28/2016, 1:22:59 AM

        Vector2 position = b2body.getPosition();
        world.destroyBody(b2body);
        BodyDef bdef = new BodyDef();
        bdef.position.set(position);
        bdef.type = BodyDef.BodyType.DynamicBody;
        b2body = world.createBody(bdef);
        FixtureDef fdef = new FixtureDef();
        CircleShape shape = new CircleShape();
        shape.setRadius(6 / MarioBros.PPM);
        fdef.filter.categoryBits = MarioBros.MARIO_BIT;
        fdef.filter.maskBits = MarioBros.GROUND_BIT |
                MarioBros.COIN_BIT |
                MarioBros.BRICK_BIT |
                MarioBros.ENEMY_BIT |
                MarioBros.OBJECT_BIT |
                MarioBros.ENEMY_HEAD_BIT |
                MarioBros.ITEM_BIT;
        fdef.shape = shape;
        b2body.createFixture(fdef).setUserData(this);
 dddpower Modified 12/28/2016, 1:16:40 AM

        EdgeShape head = new EdgeShape();
 dddpower Modified 12/26/2016, 11:52:48 PM

        head.set(new Vector2(-2 / MarioBros.PPM, 6 / MarioBros.PPM), new Vector2(2 / MarioBros.PPM, 6 / MarioBros.PPM));
        fdef.filter.categoryBits = MarioBros.MARIO_HEAD_BIT;
        fdef.shape = head;
        fdef.isSensor = true;
        b2body.createFixture(fdef).setUserData(this);
        timeToRedefineMario = false;
    }
    public void defineBigMario(){
        Vector2 currentPosition = b2body.getPosition();
        world.destroyBody(b2body);
        BodyDef bdef = new BodyDef();
        bdef.position.set(currentPosition.add(0, 10 / MarioBros.PPM));
        bdef.type = BodyDef.BodyType.DynamicBody;
        b2body = world.createBody(bdef);
        FixtureDef fdef = new FixtureDef();
        CircleShape shape = new CircleShape();
        shape.setRadius(6 / MarioBros.PPM);
        fdef.filter.categoryBits = MarioBros.MARIO_BIT;
        fdef.filter.maskBits = MarioBros.GROUND_BIT |
                MarioBros.COIN_BIT |
                MarioBros.BRICK_BIT |
                MarioBros.ENEMY_BIT |
                MarioBros.OBJECT_BIT |
                MarioBros.ENEMY_HEAD_BIT |
                MarioBros.ITEM_BIT;
        fdef.shape = shape;
        b2body.createFixture(fdef).setUserData(this);
        shape.setPosition(new Vector2(0, -14 / MarioBros.PPM));
        b2body.createFixture(fdef).setUserData(this);
        EdgeShape head = new EdgeShape();
        head.set(new Vector2(-2 / MarioBros.PPM, 6 / MarioBros.PPM), new Vector2(2 / MarioBros.PPM, 6 / MarioBros.PPM));
        fdef.filter.categoryBits = MarioBros.MARIO_HEAD_BIT;
        fdef.shape = head;
        fdef.isSensor = true;
        b2body.createFixture(fdef).setUserData(this);
        timeToDefineBigMario = false;
    }
    public void defineMario(){
 dddpower Modified 12/26/2016, 12:09:09 AM

        BodyDef bdef = new BodyDef();
        bdef.position.set(32 / MarioBros.PPM, 32 / MarioBros.PPM);
        bdef.type = BodyDef.BodyType.DynamicBody;
        b2body = world.createBody(bdef);
        FixtureDef fdef = new FixtureDef();
        CircleShape shape = new CircleShape();
        shape.setRadius(6 / MarioBros.PPM);
        fdef.filter.categoryBits = MarioBros.MARIO_BIT;
        fdef.filter.maskBits = MarioBros.GROUND_BIT |
                MarioBros.COIN_BIT |
                MarioBros.BRICK_BIT |
                MarioBros.ENEMY_BIT |
                MarioBros.OBJECT_BIT |
                MarioBros.ENEMY_HEAD_BIT |
                MarioBros.ITEM_BIT;
        fdef.shape = shape;
 dddpower Modified 12/27/2016, 8:20:33 PM

        b2body.createFixture(fdef).setUserData(this);
        EdgeShape head = new EdgeShape();
 pumpy7 Modified 10/28/2016, 3:35:19 PM
EdgeShape basically creates an edge by drawing a line between two points.
        head.set(new Vector2(-2 / MarioBros.PPM, 6 / MarioBros.PPM), new Vector2(2 / MarioBros.PPM, 6 / MarioBros.PPM));
 pumpy7 Modified 10/28/2016, 3:39:00 PM
defines his head to be an edge.
From -2 to 2 in x axis, 0 to 6 in y axis.
the origin is Mario's head
        fdef.filter.categoryBits = MarioBros.MARIO_HEAD_BIT;
        fdef.shape = head;
        fdef.isSensor = true;
 pumpy7 Modified 10/28/2016, 3:39:46 PM
When it is a sensor, it doesn't collide with objects.
        b2body.createFixture(fdef).setUserData(this);
    }
    public void fire(){
        fireballs.add(new FireBall(screen, b2body.getPosition().x, b2body.getPosition().y, runningRight ? true : false));
 73.0.145.2 Modified 2/19/2017, 1:51:41 AM
lol this sucks
    }
    public void draw(Batch batch){
        super.draw(batch);


        for(FireBall ball : fireballs)
            ball.draw(batch);
    }
}
