 //定义属性
 HorizonLine.dimensions = {
 4     WIDTH:600,    //宽600
 5     HEIGHT:12,    //高12像素
 6     YPOS:127    //在canvas中的位置
 7 };
 8 
 9 var spriteDefinition = {
10     HORIZON: {x: 2, y: 54}//地面在雪碧图中的位置
11 };
12 
13 
14 
15 /**
16 * canvas 地面将绘制到此画布上
17 * spritePos 地面在雪碧图中的坐标
18 */
19 function HorizonLine(canvas,spritePos) {
20     this.spritePos = spritePos;
21     this.canvas = canvas;
22     this.ctx = canvas.getContext("2d");
23     this.dimensions = HorizonLine.dimensions;
24 
25     //在雪碧图中坐标为2和602处分别为不同的地形
26     this.sourceXPos = [this.spritePos.x,this.spritePos.x + this.dimensions.WIDTH];
27 
28     this.xPos = []; //地面在画布中的x坐标
29     this.yPos = 0;  //地面在画布中的y坐标
30 
31     this.bumpThreshold = 0.5;    //随机地形系数
32 
33     this.setSourceDimesions();
34     this.draw();
35 }
再来看看HorizonLine原型链中的方法：

 1 HorizonLine.prototype = {
 2     setSourceDimesions:function() {
 3         //地面在画布上的位置
 4         this.xPos = [0,this.dimensions.WIDTH];//0,600
 5         this.yPos = this.dimensions.YPOS;
 6     },
 7     //随机地形
 8     getRandomType:function() {
 9         //返回第一段地形或者第二段地形
10         return Math.random() > this.bumpThreshold ? this.dimensions.WIDTH : 0;
11     },
12     draw:function() {
13         //使用9参数的drawImage方法
14         this.ctx.drawImage(imgSprite,
15             this.sourceXPos[0], this.spritePos.y,
16             this.dimensions.WIDTH, this.dimensions.HEIGHT,
17             this.xPos[0],this.yPos,
18             this.dimensions.WIDTH,this.dimensions.HEIGHT);
19 
20         this.ctx.drawImage(imgSprite,
21             this.sourceXPos[1], this.spritePos.y,
22             this.dimensions.WIDTH, this.dimensions.HEIGHT,
23             this.xPos[1],this.yPos,
24             this.dimensions.WIDTH,this.dimensions.HEIGHT);
25     },
26     updateXPos:function(pos,increment) {
27         var line1 = pos,
28             line2 = pos === 0 ? 1 : 0;
29 
30         this.xPos[line1] -= increment;
31         this.xPos[line2] = this.xPos[line1] + this.dimensions.WIDTH;
32 
33         //若第一段地面完全移出canvas外
34         if(this.xPos[line1] <= -this.dimensions.WIDTH) {
35             //则将其移动至canvas外右侧
36             this.xPos[line1] += this.dimensions.WIDTH * 2;
37             //同时将第二段地面移动至canvas内
38             this.xPos[line2] = this.xPos[line1] - this.dimensions.WIDTH;
39 
40             //选择随机地形
41             this.sourceXPos[line1] = this.getRandomType() + this.spritePos.x;
42         }
43     },
44     update:function(deltaTime,speed) {
45         var increment = Math.floor(speed * (FPS / 1000) * deltaTime);
46 
47         if(this.xPos[0] <= 0) {//交换地面一和二
48             this.updateXPos(0, increment);
49         } else {
50             this.updateXPos(1, increment);
51         }
52         this.draw();
53     },
54     reset:function() {
55         this.xPos[0] = 0;
56         this.xPos[1] = this.dimensions.WIDTH;
57     }
58 };
