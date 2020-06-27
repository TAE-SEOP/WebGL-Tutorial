# WebGL-Tutorial
## Multiple Images Texture Mapping
### 201620896 윤태섭
# 1. 프로젝트의 주제
webGL에서 정육면체의 한 면마다 다른 명화의 이미지를 넣어본다.

# 2. texture mapping
texture mappingd이란?

컴퓨터 그래픽에서 가상의 3차원 물체의 표면을 2차원의 그림이나 수식으로 적용하여 실제 물체처럼 느껴지도록 하는 기법이다.

UV-mapping : u와 v를 사용하여 2차원의 그림을 3차원의 모델로 만드는 프로세스이다.


# 3. 방법
6개의 이미지를 정육면체의 한 면마다 적용하기 위해서 6가지의 texture를 각각 reference할 복잡한 shader를 만들고 fragment shader에 vertex당 사용할 texture를 결정하는 방법이 있다.
하지만 이 방법은 굉장히 복잡한 방식이 될 것이다. 조금 더 간단한 방법이 있다.
6개의 이미지를 하나의 texture에 넣는 방식이다. 6개의 명화를 하나의 texture에 넣으면 이런 그림이 된다.

![painting](/uploads/dea127c2d8d4ad11b966785ffe5f2e01/painting.png)

이미지는 저장될 때 좌에서 우로, 위에서 아래로 저장되기 때문에 실제 작업을 할 때는 

![painting_reflect](/uploads/047d625d79d1aac385145ec8652917b3/painting_reflect.png)

이런 모양을 하고 있다고 생각해야 한다. 
이 texture를 texture corrdinates로 보면 아래의 그림처럼 표현된다.

![texturecoord](/uploads/f8c46bd21a9846a2fff46dec0b624dcb/texturecoord.PNG)

따라서 각 u,v를 통해 그림을 넣기 위해서는 위의 사항들을 고려햐여 결정해야 한다.  
위의 사항들을 고려하여 정육면체의 각 면에 각각의 명화를 texture mapping한다.

![cube1](/uploads/dab3f0276736be3f5a8ea3e4bd5e5543/cube1.PNG)  ![cube2](/uploads/72fab0216b7589bad520e3c9f9848196/cube2.PNG)

큐브를 마우스로 클릭하고 돌리면 큐브가 돌아가서 다른 면들을 볼 수 있다.


# 4. 코드


    var texture = gl.createTexture();
	gl.bindTexture(gl.TEXTURE_2D, texture);
    var image = new Image();
    image.src = "넣을 이미지";
    image.addEventListener('load', function() {
		// Now that the image has loaded make copy it to the texture.
		gl.bindTexture(gl.TEXTURE_2D, texture);
		gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA,gl.UNSIGNED_BYTE, image);
		gl.generateMipmap(gl.TEXTURE_2D);
        });
        
정육면체를 구성하는 vertex중에서 모나리자를 그릴 면을 예시로 보면 texture coordinates에서 모나리자는 x축 0.0~ 1/3, y축 0.0 ~ 1/2에 위치해 있다. 따라서 그리는 삼각형에 맞춰
해당 uv값을 결정하면 된다. 

         //모나리자 , sx, sy, sz = 1  , au1=0.0, au2=1/3, av1=0.0, av2=1/2 
        -sx/2, -sy/2, sz/2, 0.1, 0.9, 1.0, 1.0,    au1, av1,
        -sx/2,  sy/2, sz/2, 1.0, 0.1, 1.0, 1.0,    au2, av1,
         sx/2,  sy/2, sz/2,  1.0, 1.0, 0.1, 1.0,   au2, av2, 
         -sx/2, -sy/2, sz/2, 0.2, 0.8, 1.0, 1.0,   au1, av1, 
         sx/2, -sy/2, sz/2, 1.0, 0.2, 0.0, 1.0,    au1, av2,
         sx/2,  sy/2, sz/2,  1.0, 1.0, 0.2, 1.0,   au2, av2, 
    
fragment shader에서 정육면체의 입혀질 color를 texture로 바꾸기 위해 gl_FragColor = 0.0 * color + 1.0 * texture2D(sampler2d, texCoord)로 한다. 

    var fragmentShaderSource = '\
            varying highp vec4 color; \
            varying mediump vec2 texCoord;\
            uniform sampler2D sampler2d; \
			void main(void) \
			{ \
                gl_FragColor = 0.0 * color + 1.0 * texture2D(sampler2d, texCoord); \
			}';
        
vertex shader에 texCoord = myUV 를 넣는다. 

    var vertexShaderSource = '\
			attribute highp vec4 myVertex; \
            attribute highp vec4 myColor; \
            attribute highp vec2 myUV; \
			uniform mediump mat4 mmat; \
			uniform mediump mat4 vmat; \
			uniform mediump mat4 pmat; \
            varying highp vec4 color;\
            varying mediump vec2 texCoord;\
			void main(void)  \
			{ \
                gl_Position = pmat*vmat*mmat * myVertex; \
                color = myColor; \
                texCoord = myUV; \
			}';
전역 변수로 지정하여 움직임을 컨트롤할 수 있게 한다.	

    var drag = false;
    var old_x, old_y;
    var dX = 0, dY = 0;

마우스로 인해 발생하는 이벤트를 처리하는 함수이다. 
canvas에서 마우스가 움직이는 것을 dX,dY로 계산하여 rotX,rotY에 더해주어 큐브가 이벤트에 맞게 움직이도록 한다. 
    var mouseDown = function(e) {
       drag = true;
       old_x = e.pageX, old_y = e.pageY;
       e.preventDefault();
       return false;
    };

    var mouseUp = function(e){
       drag = false;
    };

    var mouseMove = function(e) {
       if (!drag) return false;
            dX = (e.pageX-old_x)*2*Math.PI/canvas.width;
            dY = (e.pageY-old_y)*2*Math.PI/canvas.height;
            rotX+= dX;
            rotY+= dY;
            old_x = e.pageX, old_y = e.pageY;
            e.preventDefault();
       
    };
    
계속돌아가는 것을 방지하기 위해 drag가 아닌 상태와 큐브가 움직이고 있다면 rotX,rotY를 1.1로 나누어 속도가 줄어들게 한다. 
    
    if (!drag && (rotX != 0.0 || rotY !=0.0)) {
    rotX = rotX/1.1;
    rotY = rotY/1.1;
    }
 

			
# Reference
[https://webglfundamentals.org/webgl/lessons/webgl-3d-textures.html](https://webglfundamentals.org/webgl/lessons/webgl-3d-textures.html)

[https://git.ajou.ac.kr/hwan/webgl-tutorial/-/tree/master/student2019/201520868](https://git.ajou.ac.kr/hwan/webgl-tutorial/-/tree/master/student2019/201520868)

[https://git.ajou.ac.kr/hwan/cg_course/-/tree/master/WebGL/texture](https://git.ajou.ac.kr/hwan/cg_course/-/tree/master/WebGL/texture)

[https://git.ajou.ac.kr/hwan/webgl-tutorial/-/tree/master/student2019/201421044](https://git.ajou.ac.kr/hwan/webgl-tutorial/-/tree/master/student2019/201421044)

[https://www.tutorialspoint.com/webgl/webgl_interactive_cube.htm](https://www.tutorialspoint.com/webgl/webgl_interactive_cube.htm)

# 저작권
저작권은 원칙적으로 저작자 사후 70년까지 보호되며, 2013년 7월 1일 시행 이전에 보호 기간이 만료된 저작물의 기간은 사후 50년간 존속한다. 

프로젝트에서 사용한 명화들은 전부 저작자 사후 70년을 넘은 저작권이 만료된 명화들이다. 


