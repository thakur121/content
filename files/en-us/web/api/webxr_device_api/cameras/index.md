---
title: 'Viewpoints and viewers: Simulating cameras in WebXR'
slug: Web/API/WebXR_Device_API/Cameras
tags:
  - 3D
  - API
  - AR
  - Advanced
  - Cameras
  - Coordinates
  - Guide
  - Location
  - Motion
  - Position
  - VR
  - Viewers
  - Viewpoints
  - WebGL
  - WebXR
  - WebXR API
  - WebXR Device API
  - XR
  - XRPose
  - XRView
  - XRViewerPose
  - rotation
  - transform
---
<p>{{DefaultAPISidebar("WebXR Device API")}}</p>

<p>The first and most important thing to understand when considering the code to manage point-of-view and cameras in your application is this: <em>WebXR does not have cameras</em>. There's no magic object provided by either the <a href="/en-US/docs/Web/API/WebGL_API">WebGL</a> or the <a href="/en-US/docs/Web/API/WebXR_Device_API">WebXR</a> API that represents the viewer that you can rotate and move around to automatically change what's seen on the screen. <span class="seoSummary">In this guide we show how use <a href="/en-US/docs/Web/API/WebGL_API">WebGL</a> to simulate camera movements without having a camera to move. These techniques can be used in any WebGL (or WebXR) project.</span></p>

<p>Animating 3D graphics is an area of software development that brings together multiple disciplines in computer science, mathematics, art, graphic design, kinematics, anatomy, physiology, physics, and cinematography. Since we don't have a real camera, we imagine one, reproducing the <em>effect</em> of having a camera, without actually having the ability to move the user around the scene.</p>

<p>There are a few articles about the fundamental math, geometry, and other concepts behind WebGL and WebXR which may be useful to read before or while reading this one, including:</p>

<ul>
 <li><a href="/en-US/docs/Games/Techniques/3D_on_the_web/Basic_theory">Explaining basic 3D theory</a></li>
 <li><a href="/en-US/docs/Web/API/WebGL_API/Matrix_math_for_the_web">Matrix math for the web</a></li>
 <li><a href="/en-US/docs/Web/API/WebGL_API/WebGL_model_view_projection">WebGL model view projection</a></li>
 <li><a href="/en-US/docs/Web/API/WebXR_Device_API/Geometry">Geometry and reference spaces in WebXR</a></li>
</ul>

<p><em>Ed. note: Most diagrams used in this article to show how the camera moves while performing standard movements were taken from <a href="https://filmmakeriq.com/2016/09/the-importance-and-not-so-importance-of-film-terminology/">an article on the FilmmakerIQ web site</a>; namely, from <a href="https://filmmakeriq.com/wp-content/uploads/2016/09/Pan-Tilt.png">this image</a> which is found all over the web. We assume due to their frequent reuse that they're available under a permissive license, ownership is not certain. We hope that it's freely usable; if not, and you're the owner, please let us know and we'll find or produce new diagrams. Or, if you're happy to let us continue to use the images, please let us know so we can credit you properly!</em></p>

<h2 id="Cameras_and_relative_movement">Cameras and relative movement</h2>

<p>When a classic live-action movie is filmed, the actors are on a set and move about the set as they perform, with one or more cameras watching their moves. The cameras may be fixed in place, but they may also be set up to move around as well, tracking the movement of the performers, dollying in and out to achieve emotional impact, and so forth.</p>

<h3 id="Virtual_cameras">Virtual cameras</h3>

<p>In WebGL (and by extension, in WebXR), there is no camera object we can move and rotate, so we have to find a way to fake these movements. Since there is no camera, we have to find a way to fake it. Fortunately, physicists like Galileo, Newton, Lorentz, and Einstein have given us the <strong>{{interwiki("wikipedia", "principle of relativity")}}</strong>, which states that the laws of physics have the same form in every frame of reference. That is, no matter where you're standing, the laws of physics work in the same way.</p>

<p>By extension, if you and another person are standing in an empty field of solid stone with nothing else visible as far as the eye can see, if you move three meters toward the other person, the result <em>looks the same</em> as if the other person had moved three meters toward you. There is no way for either of you to see the difference. A third party can tell the difference, but the two of you cannot. If you are a camera, you can achieve the same visual result both by moving the camera <em>or by moving everything around the camera</em>.</p>

<p>And that's our solution. Since we can't move the camera, we move the world around it. Our renderer needs to know where we imagine the camera to be, then alter the position of every visible object to simulate that position and orientation. Thus, instead of referring to an actual camera object, the term <strong>camera</strong> is used in WebGL and WebXR programming to refer to an object describing the position and viewing direction of a hypothetical viewer of the scene, whether there's an actual object present in 3D space or not.</p>

<h3 id="Points_of_view">Points of view</h3>

<p>Since the camera is a virtual object which, rather than necessarily representing a physical object in the virtual world, represents a viewer's position and viewing direction, it's useful to think about the kinds of situation that call for the use of a camera. Gaming-related situations are listed separately since they are often a special case specific to gaming, but any of these perspectives might apply to any 3D graphics scene.</p>

<h4 id="Generalized_cameras">Generalized cameras</h4>

<p>In general, virtual cameras may or may not be incorporated into physical objects within the scene. Indeed, outside the scope of 3D gaming, the odds are far more likely that the camera will not correspond with an object that appears in the scene at all. Some examples of ways 3D cameras are used:</p>

<ul>
 <li>When rendering animation—whether for filmmaking or for use within the context of a presentation or game—the virtual camera is used just like a real-world film camera. As much as possible, <a href="#simulating_classic_cinematography">standard cinematographic techniques</a> are used, since the viewer has likely grown up watching films using those techniques, and has subconscious expectations that a film or animation will follow those methods. Deviating from them can pull the viewer out of the moment.</li>
 <li>In business applications, the 3D camera is used to set the apparent size and perspective when rendering things such as graphs and charts.</li>
 <li>In mapping applications, the camera may either be placed directly over the scene, or might use various angles to show perspective. For 3D GPS solutions, the camera is positioned to show the area around the user, with the majority of the display showing the area ahead of the user's path of motion.</li>
 <li>When using WebGL to accelerate 2D graphics drawing, the camera is typically placed directly above the center of the scene with the distance and field of view set to allow the entire scene to be presented.</li>
 <li>When accelerating bitmapped graphics, the renderer would draw the 2D image into a WebGL texture's buffer, then redraw the texture to refresh the screen. This essentially uses the texture as a backbuffer for performing {{interwiki("wikipedia", "multiple buffering")}} in your 2D graphics application.</li>
</ul>

<h4 id="Cameras_in_gaming">Cameras in gaming</h4>

<p>There are many kinds of games, and as such, there are several ways in which cameras might be used in games. Some common situations include:</p>

<ul>
 <li>In a first-person game, the camera is located within the player's avatar's head, facing in the same direction as the avatar's eyes. This way, the view presented on the player's screen or headset is what their avatar would be seeing.</li>
 <li>In some third-person games, the camera is located a short distance behind the player's avatar or vehicle, showing them from behind as they move through the game world. This is used in a lot of multiplayer online role-playing games, certain shooter games, and so forth. Popular examples include <em>World of Warcraft</em>, <em>Tomb Raider</em>, and <em>Fortnite</em>. This category also includes games where the camera is placed just over the player's shoulder.</li>
 <li>Some 3D games offer the ability to change your point of view, such as to look out the various windows of an aircraft in a flight simulator, or to see the views from all the security cameras within the game level (a common feature of spy and stealth-based games). This ability is also used by games offering weapons with scopes, where the view isn't quite based off the head's position in the same way anymore.</li>
 <li>3D games might also provide the ability for non-players to observe the action, either by positioning an invisible avatar of sorts or by choosing a fixed virtual camera to watch from.</li>
 <li>In advanced 3D games, a camera or camera-like object <em>could</em> be used to determine what a non-player character can see, relying on the same rendering and physics engine used by player characters for non-player characters.</li>
 <li>In single-screen 2D games, the camera is not associated directly with the player or any other character in the game, but is instead either fixed above or next to the game play area, or follows the action as the action moves around a scrolling game world. For example, a classic arcade game such as <em>Pac-Man</em> takes place on a fixed game map, so the camera remains fixed a set distance above the map, always pointed straight down at the game world.</li>
 <li>In a side-scrolling or top-scrolling game such as <em>Super Mario Bros.</em>, the camera moves along left and right (or up and down, or both) to ensure that the action remains visible even though the game level is much larger than the viewport.</li>
</ul>

<h3 id="Positioning_the_camera">Positioning the camera</h3>

<p>Since there are no standard camera objects in WebGL or WebXR, we need to simulate the camera ourselves. Before we can do so, and before we can then simulate the movement of the camera, let's actually take a look at the virtual camera and how it <em>can</em> move, at the most fundamental level. As in all things, the <strong>position</strong> of an object in space—even if that space if virtual—can be represented using three numbers that indicate its position relative to the origin, whose position is defined to be (0, 0, 0).</p>

<p>There's another aspect of the spatial relationship of an object to the origin in space left to be considered: <strong>perspective</strong>. Perspective, properly applied to the objects in a scene, can take a scene that would otherwise look as flat as a typical 2D screen and make it pop as if it were truly 3D. There are several kinds of perspective; those are defined and their mathematics explained in the article <a href="/en-US/docs/Web/API/WebGL_API/WebGL_model_view_projection">WebGL model view projection</a>. Importantly, the effect of perspective on a vector can be represented by adding a fourth component to the vector: the perspective component, called <code>w</code>.</p>

<p>The value of <code>w</code> is applied by dividing each of the other three components by it to get the final position or vector; that is, for a coordinate given as (<code>x</code>, <code>y</code>, <code>z</code>, <code>w</code>), the point in the 3D space is actually (<code>x</code>/<code>w</code>, <code>y</code>/<code>w</code>, <code>z</code>/<code>w</code>, 1) or (<code>x</code>/<code>w</code>, <code>y</code>/<code>w</code>, <code>z</code>/<code>w</code>). If you're not using perspective, <code>w</code> is always 1. In that situation, the complete coordinates for an object located at (1, 0, 3) are (1, 0, 3, 1).</p>

<p>But the location isn't enough to describe an object in 3D space because an object's state in space isn't only about its location, but about its rotation or facing direction, otherwise known as its <strong>orientation</strong>. The orientation can be represented using a 3D vector, which is typically normalized so that its length is 1.0. For example, if the object is facing an object located at (3, 1, -2)—that is, three meters to the right, one meter up, and two meters away from the origin point—the result is:</p>

<p><math display="block"><semantics><mrow><mo>[</mo><mtable rowspacing="0.5ex"><mtr><mtd><mn>3</mn></mtd></mtr><mtr><mtd><mn>1</mn></mtd></mtr><mtr><mtd><mo>-</mo><mn>2</mn></mtd></mtr></mtable><mo>]</mo></mrow><annotation encoding="TeX">\left [ \begin{matrix} 3 \\ 1 \\ -2 \end{matrix} \right ] </annotation></semantics></math></p>

<p>This can also be represented as an array:</p>

<pre class="brush: js">let directionVector = [3, 1, -2];
</pre>

<p>For the purposes of performing operations involving both the coordinates and the facing direction vector, the vector needs to include the <code>w</code> component. The value of <code>w</code> is always 0 for vectors, so the aforementioned vector can also be represented using <code>[3, 1, -2, 0]</code> or:</p>

<p><math display="block"><semantics><mrow><mo>[</mo><mtable rowspacing="0.5ex"><mtr><mtd><mn>3</mn></mtd></mtr><mtr><mtd><mn>1</mn></mtd></mtr><mtr><mtd><mo>-</mo><mn>2</mn></mtd></mtr><mtr><mtd><mn>0</mn></mtd></mtr></mtable><mo>]</mo></mrow><annotation encoding="TeX">\left [ \begin{matrix} 3 \\ 1 \\ -2 \\ 0 \end{matrix} \right ] </annotation></semantics></math></p>

<p>WebXR automatically normalizes vectors to have a length of 1 meter; however, you may find that it makes sense to do it yourself for various reasons, such as to improve performance of calculations by not having to repeatedly perform normalization.</p>

<p>Once you've determined the matrix representing the combination of movements you wish the camera to make, you need to reverse it, because you're not moving the camera. Since you're actually moving everything <em>except</em> the camera, take the inverse of the transform matrix to get an inverse transform matrix. This inverse matrix can then be applied to the objects in the world to alter their positions and orientations to simulate the desired camera position.</p>

<p>This is why the {{domxref("XRRigidTransform")}} object used by WebXR to represent transforms includes an {{domxref("XRRigidTransform.inverse", "inverse")}} property. The <code>inverse</code> property is another <code>XRRigidTransform</code> object which is the inverse of the parent transform. Since the {{domxref("XRView")}} representing the view has a {{domxref("XRView.transform", "transform")}} property which is an <code>XRRigidTransform</code> providing the camera view, you can get the model view matrix—the transform matrix needed to move the world to simulate the desired camera position—like this:</p>

<pre class="brush: js">let viewMatrix = view.transform.inverse.matrix;</pre>

<p>If the library you're using accepts an <code>XRRigidTransform</code> object directly, you can instead get <code>view.transform.inverse</code>, rather than pulling out just the array representing the view matrix.</p>

<h3 id="Composing_multiple_transforms">Composing multiple transforms</h3>

<p>If your camera needs to be performing multiple transforms simultaneously, such as zooming and panning at the same time, you can multiply the transform matrices together to compose them into a single matrix that applies both changes at once. See <a href="/en-US/docs/Web/API/WebGL_API/Matrix_math_for_the_web#multiplying_two_matrices">Multiplying two matrices</a> in the article <a href="/en-US/docs/Web/API/WebGL_API/Matrix_math_for_the_web">Matrix math for the web</a> for a clear but readable function that does this or use your preferred matrix math library such as <a href="https://glmatrix.net/">glMatrix</a> to do the work.</p>

<p>It's crucial to remember that unlike typical arithmetic, where multiplication is commutative (that is, you get the same answer whether you multiply left to right or right to left), matrix multiplication <em>is not commutative!</em> This is because each transform affects the position of the object and possibly the very coordinate system itself, which can dramatically change the results of the next operation performed. So you need to be careful about the order in which you apply your transforms when building your composite transform (or directly applying transforms in sequence).</p>

<h3 id="Applying_the_transform">Applying the transform</h3>

<p>To apply the transform, you multiply the point or vector by the transform or composition of transforms.</p>

<p>This has been a very quick overview of the concepts of position in terms of physical location, orientation or facing direction, and perspective. For more detail on the subject, see the articles <a href="/en-US/docs/Web/API/WebXR_Device_API/Geometry">Geometry and reference spaces</a>, <a href="/en-US/docs/Web/API/WebGL_API/WebGL_model_view_projection">WebGL model view projection</a>, and <a href="/en-US/docs/Web/API/WebGL_API/Matrix_math_for_the_web">Matrix math for the web</a>.</p>

<h2 id="Simulating_classic_cinematography">Simulating classic cinematography</h2>

<p>Cinematography is the art of designing, planning, and executing camera movements to create the desired look and emotion for a scene in animation or film. There are a number of terms that are helpful to understand, primarily around camera movement, as these terms are used to describe designed viewpoint changes with the virtual camera. It's also entirely possible to perform more than one of these movements at the same time; for example, you can pan the camera while also zooming in on the scene.</p>

<p>Keep in mind that the majority of camera movements are described relative to the camera's reference space.</p>

<p>The format for storing matrices is generally as a flat array in column-major order; that is, the values from the matrix are written starting with the top-left corner and moving <em>down</em> to the bottom, then moving over to the right a row and repeating until all values are in the array.</p>

<p>Thus a matrix that looks like this:</p>

<p><math display="block"><semantics><mrow><mo>[</mo><mtable rowspacing="0.5ex"><mtr><mtd><msub><mi>a</mi><mn>1</mn></msub></mtd><mtd><msub><mi>a</mi><mn>5</mn></msub></mtd><mtd><msub><mi>a</mi><mn>9</mn></msub></mtd><mtd><msub><mi>a</mi><mn>13</mn></msub></mtd></mtr><mtr><mtd><msub><mi>a</mi><mn>2</mn></msub></mtd><mtd><msub><mi>a</mi><mn>6</mn></msub></mtd><mtd><msub><mi>a</mi><mn>10</mn></msub></mtd><mtd><msub><mi>a</mi><mn>14</mn></msub></mtd></mtr><mtr><mtd><msub><mi>a</mi><mn>3</mn></msub></mtd><mtd><msub><mi>a</mi><mn>7</mn></msub></mtd><mtd><msub><mi>a</mi><mn>11</mn></msub></mtd><mtd><msub><mi>a</mi><mn>15</mn></msub></mtd></mtr><mtr><mtd><msub><mi>a</mi><mn>4</mn></msub></mtd><mtd><msub><mi>a</mi><mn>8</mn></msub></mtd><mtd><msub><mi>a</mi><mn>12</mn></msub></mtd><mtd><msub><mi>a</mi><mn>16</mn></msub></mtd></mtr></mtable><mo>]</mo></mrow><annotation encoding="TeX">\left [ \begin{matrix} a_{1} &amp; a_{5} &amp; a_{9} &amp; a_{13} \\ a_{2} &amp; a_{6} &amp; a_{10} &amp; a_{14} \\ a_{3} &amp; a_{7} &amp; a_{11} &amp; a_{15} \\ a_{4} &amp; a_{8} &amp; a_{12} &amp; a_{16} \end{matrix} \right ]</annotation></semantics></math></p>

<p>Is represented in array form like this:</p>

<pre class="brush: js">let matrixArray = [a1, a2, a3, a4, a5, a6, a7, a8,
                   a9, a10, a11, a12, a13, a14, a15, a16];</pre>

<p>In this array, the leftmost column contains the entries <em>a</em>₁, <em>a</em>₂, <em>a</em>₃, and <em>a</em>₄. The topmost row contains the entries <em>a</em>₁, <em>a</em>₅, <em>a</em>₉, and <em>a</em>₁₃</sub>.</p>

<p>Keep in mind that most WebGL and WebXR programming is done using third-party libraries which expand upon the basic functionality of WebGL by adding routines that make it much easier to perform not only core matrix and other operations, but often also to simulate these standard cinematography techniques. You should strongly consider using one instead of directly using WebGL. This guide uses WebGL directly since it's useful to understand to some extent what goes on under the hood, and to aide in the development of libraries or to help you optimize code.</p>

<div class="notecard note">
<p><strong>Note:</strong> Even though we use phrases like "move the camera," what we're really doing is moving the entire world around the camera. This affects the way certain values work, which will be noted as they come up below.</p>
</div>

<h3 id="Zooming">Zooming</h3>

<p>Among the best known camera effects is the <strong>zoom</strong>. Zooming is performed in a physical camera by altering the focal length of the lens; this is the distance between the center of the lens itself and the camera's light sensors. Thus, zooming doesn't actually involve moving the camera at all. Instead, a zoom shot changes the magnification of the camera over time to make the area of focus seem closer to or farther away from the viewer, without actually physically moving the camera. A slow move can bring a sense of movement, ease, or focus to a scene, while a rapid zoom can create a sense of anxiety, surprise, or tension.</p>

<p>Because a zoom does not move the camera's position, the resulting effect is unnatural. The human eye doesn't have a zoom lens on it. We make things smaller or larger by moving away from or toward them. In cinematography, that's called a <a href="#dollying_moving_in_or_out">dolly shot</a>.</p>

<p>There are two techniques in 3D graphics that can create similar though not identical results, and whose methods apply more easily in different situations.</p>

<h4 id="Zooming_by_adjusting_field_of_view">Zooming by adjusting field of view</h4>

<p>You can do something more akin to a true "zoom" by altering the camera's <strong>field of view</strong> (<strong>FOV</strong>). The field of view is an angle defining the length of the arc on the entire viewable area surrounding the camera that should be visible at once. This is an effect of the focal length in a physical camera, so since there is no true camera, altering the FOV is a passable substitute.</p>

<p>Recall that the circumference of a circle is 2π⋅r radians (360°); as such, this is the theoretical maximum FOV. Realistically, though, not only do humans not see anywhere near that much, but viewing devices such as monitors and VR goggles tend to reduce the field of view even further. Human eyes typically have a horizontal field of view of around 135° (about 2.356 radians) and a vertical FOV of about 180° (π or around 3.142 radians).</p>

<p>Making the camera's FOV smaller reduces the arc that will be included in the viewport, thus enlarging that content when rendered to the view. There are differences between this and an optical zoom effect, but the result is generally close enough to get the job done.</p>

<p>The following function returns a projection perspective matrix that integrates the specified field of view angle as well as the given near and far clipping plane distances:</p>

<pre class="brush: js">function createPerspectiveMatrix(viewport, fovDegrees, nearClip, farClip) {
  const fovRadians = fovDegrees * (Math.PI / 180.0);
  const aspectRatio = viewport.width / viewport.height;

  const transform = mat4.create();
  mat4.perspective(transform, fovRadians, aspectRatio,
                   nearClip, farClip);
  return transform;
}</pre>

<p>After converting the FOV angle, <code>fovDegrees</code>, from degrees to radians and computing the aspect ratio of the {{domxref("XRViewport")}} specified by the <code>viewport</code> parameter, this function uses the <a href="https://glmatrix.net/">glMatrix</a> library's <code><a href="https://glmatrix.net/docs/module-mat4.html#.perspective">mat4.perspective()</a></code> function to compute the perspective matrix.</p>

<p>The perspective matrix encapsulates the field of view (technically, this is the <em>vertical</em> field of view), aspect ratio, and the near and far clipping planes within the 4x4 matrix <code>transform</code>, which is then returned to the caller.</p>

<p>The near clipping plane is the distance in meters to a plane parallel to the display surface closer than which nothing gets drawn. Any vertices which lie on the same side of that plane as the camera are not drawn. Conversely, the far clipping plane is the distance in meters to a plane beyond which no vertices are drawn.</p>

<p>To zoom using a scaling factor or percentage, you can map 1x (100% of normal size) to the largest value of FOV you allow (which leads to the most content being visible), then map your maximum magnification to the maximum value of FOV you support and map corresponding values in between.</p>

<p>If you start each frame's rendering pass by computing the perspective matrix, you can then multply into that matrix all the other transforms you need to apply in order to result in the frame's desired geometry. For example:</p>

<pre class="brush: js">const transform = createPerspectiveMatrix(viewport, 130, 1, 100);
const translateVec = vec3.fromValues(-trackDistance, -craneDistance, pushDistance);
mat4.translate(transform, transform, translateVec);
</pre>

<p>This starts with the perspective matrix representing a 130° vertical field of view, then applies a translation that moves the camera in a manner that includes <a href="#track">track</a>, <a href="#crane">crane</a>, and <a href="#push">push</a> movements.</p>

<h4 id="Scaling_transforms">Scaling transforms</h4>

<p>Unlike a true "zoom", <strong>scaling</strong> involves multiplying each of the <code>x</code>, <code>y</code>, and <code>z</code> coordinate values in a position or vertex by a scaling factor for that axis. These may or may not necessarily be identical for each axis, though the closest result you can get to a zoom effect would involve using the same value for each. This would need to be applied to every vertex in the scene—ideally in the vertex shader.</p>

<p>If you want to scale up by a factor of 2, you need to multiply each component by 2.0. To scale down by the same amount, multiply them by -2.0. In matrix terms, this is performed using a transform matrix with scaling factored into it, like this:</p>

<pre class="brush: js">let scaleTransform = [
  Sx,  0,  0,  0,
   0, Sy,  0,  0,
   0,  0, Sz,  0,
   0,  0,  0,  1
];
</pre>

<p>This matrix represents a transform that scales up or down by a factor indicated by <code>(Sx, Sy, Sz)</code>, where <code>Sx</code> indicates the scaling factor along the X axis, <code>Sy</code> the scaling factor along the Y axis, and <code>Sz</code> the factor for the Z axis. If any of these values differs from the others, the result will be stretching or contraction which is different in some dimensions compared to others.</p>

<p>If the same scaling factor is to be applied in every direction, you can create a simple function to generate the scaling transform matrix for you:</p>

<pre class="brush: js">function createScalingMatrix(f) {
  return [
    f, 0, 0, 0,
    0, f, 0, 0,
    0, 0, f, 0,
    0, 0, 0, 1
  ];
}
</pre>

<p>With the transform matrix in hand, we apply the transform <code>scaleTransform</code> to the vector (or vertex) <code>myVector</code>:</p>

<pre class="brush: js">let myVector = [2, 1, -3];
let scaleTransform = [
  2, 0, 0, 0,
  0, 2, 0, 0,
  0, 0, 2, 0,
  0, 0, 0, 1
];
vec4.transformMat4(myVector, myVector, scaleTransform);
</pre>

<p>Or, using scaling along every axis by the same factor using the <code>createScalingMatrix()</code> function shown above:</p>

<pre class="brush: js">let myVector = [2, 1, -3];
vec4.transformMat4(myVector, myVector, createScalingMatrix(2.0));</pre>

<h3 id="Panning_Yawing_left_or_right">Panning (Yawing left or right)</h3>

<p><strong>Panning</strong> or <strong>yaw</strong> is the rotation of the camera left to right or right to left, with its base otherwise fixed in place. The position of the camera in space does not change, only the direction in which it's looking. And that direction does not change other than horizontally. Panning is great for establishing a setting or providing a sense of scope in a vast space or on a vast object. Or just for looking left and right, like simulating the player turning their head in an immersive or VR scenario.</p>

<img alt="A diagram showing a camera panning left or right" src="camera-pan.png">

<p>To do this, then, we need to rotate around the Y axis, to simulate the left and right rotation of the camera. Using the <a href="https://glmatrix.net/">glMatrix</a> library we've used previously, this can be performed using the <code>rotateY()</code> method on the <code>mat4</code> class, which represents a standard 4x4 matrix. To rotate the viewpoint defined by the matrix <code>viewMatrix</code> by <code>panAngle</code> radians:</p>

<pre class="brush: js">mat4.rotateY(viewMatrix, viewMatrix, panAngle);
</pre>

<p>If <code>panAngle</code> is positive, this transform will pan the camera to the right; a negative value for <code>panAngle</code> will pan to the left.</p>

<h3 id="Tilting_Pitching_up_or_down">Tilting (Pitching up or down)</h3>

<p>When you <strong>tilt</strong> or <strong>pitch</strong> the camera, you keep it fixed in space at the same coordinates while changing the direction in which it's facing vertically without altering the horizontal portion of its facing at all. It adjusts the direction it's pointing up and down. Tilting is good for capturing the scope of a tall object or scene, such as a forest or a mountain, but is also a popular way to introduce a character or locale of importance or which inspires awe. it's also of course useful for implementing support for a player looking up and down.</p>

<img alt="A diagram showing a camera tilting up and down" src="camera-tilt.png">

<p>Thus, tilting the camera can be achieved by rotating the camera around the X axis, so that it pivots to look up and down. This can be done using the appropriate method in your matrix math library, such as the <code>rotateX()</code> method in glMatrix's <code>mat4</code> class:</p>

<pre class="brush: js">mat4.rotateX(viewMatrix, viewMatrix, angle);</pre>

<p>Positive values for <code>angle</code> will tilt the camera downward, while negative values of <code>angle</code> will tilt upward.</p>

<h3 id="Dollying_Moving_in_or_out">Dollying (Moving in or out)</h3>

<p>A <strong>dolly</strong> shot is one in which the entire camera is moved forward and backward. In classic filmmaking, this is typically done with the camera mounted on a track or on a moving vehicle. The resulting motion can create impressively smooth effects, especially if moving along with the person or object that's the focus of your shot.</p>

<img alt="A diagram showing how a camera moves for a dolly shot" src="camera-dolly.png">

<p>While a dolly shot and a zoom seem like they ought to look about the same, they don't. The fact that zooming alters the camera's focal length means that the spatial relationship between the target and its surroundings doesn't change even as the target gets larger or smaller in the frame. On the other hand, a dolly shot, by actually moving the camera, replicates the sense of physical movement, causing the relationships of objects in the scene to shift as you expect while moving past them as you go toward or away from the target of the shot.</p>

<p>To perform a dolly operation, translate the camera view forward and backward along the Z axis:</p>

<pre class="brush: js">mat4.translate(viewMatrix, viewMatrix, [0, 0, dollyDistance]);</pre>

<p>Here, <code>[0, 0, dollyDistance]</code> is a vector wherein <code>dollyDistance</code> is the distance to dolly the camera. Since this works by moving the entire world around the camera, what really happens here is that the entire world moves along the Z axis by <code>dollyDistance</code> meters relative to the camera. If <code>dollyDistance</code> is positive, the world moves toward the user by that amount, resulting in the camera being closer to the scene. Contrariwise, negative values of <code>dollyDistance</code> move the world away from the user, causing the camera to appear to move backward from the target.</p>

<h3 id="Trucking_Moving_left_or_right">Trucking (Moving left or right)</h3>

<p><strong>Trucking</strong> using a physical camera uses the same kind of rigging as dollying, but instead of moving the camera forward and backward, it moves from left to right or vice-versa. The camera doesn't rotate at all, so the focus of the shot slowly glides off the screen. This can suggest concentration, time passing, or contemplation when attempting to establish emotion in a scene. It's also used frequently in "walk-and-talk" scenes, wherein the camera glides alongside the characters and they walk through the scene.</p>

<img alt="A diagram showing how a camera trucks left and right" src="camera-truck.png">

<p>To move the camera left and right, translate the view matrix along the X axis in the opposite direction from the desired camera movement:</p>

<pre class="brush: js">mat4.translate(viewMatrix, viewMatrix, [-truckDistance, 0, 0]);</pre>

<p>Note the vector <code>[-truckDistance, 0, 0]</code>. This compensates for the fact that the truck operation works by moving the world rather than the camera. By moving the entire world in the opposite direction from the direction indicated by <code>truckDistance</code>, we achieve the effect of moving the camera the expected direction. This way, positive values of <code>truckDistance</code> will move the camera to the right (by moving the world to the left) and negative values of <code>truckDistance</code> will move the camera to the left by moving the world to the right.</p>

<h3 id="Pedestaling_Moving_up_or_down">Pedestaling (Moving up or down)</h3>

<p>A <strong>pedestal</strong> shot is one involving keeping the camera fixed horizontally relative to the floor, but moved straight up or down. Picture the camera on a pedestal (or pole) that gets taller or shorter. This is useful when tracking a subject that's getting taller or shorter, or is standing up or sitting down from a chair, or moving straight up and down.</p>

<img alt="A diagram showing a camera moving up and down using a pedestal motion" src="camera-pedestal.png">

<p>This is similar to a <strong>crane</strong> shot, which involves moving a camera attached to a crane up and down. To perform a pedestal or crane motion, translate the view along the Y axis in the opposite direction from the direction you want to move the camera:</p>

<pre class="brush: js">mat4.translate(viewMatrix, viewMatrix, [0, -pedestalDistance, 0]);</pre>

<p>By negating the value of <code>pedestalDistance</code>, we compensate for the fact that we're actually moving the world rather than the camera. So positive values of <code>pedestalDistance</code> will move the camera up, while negative values will move it down.</p>

<h3 id="Canting_Rolling_left_and_right">Canting (Rolling left and right)</h3>

<p><strong>Cant</strong> (or <strong>roll</strong>) is a rotation of the camera around its roll axis; that is, the camera remains fixed in space, and remains pointed at the same location, but rotates around so that the top of the camera is pointed in a different direction.</p>

<img alt="A diagram showing a camera rolling left and right" src="camera-roll.png">

<p>You can visualize this by holding your arm out in front of you with your hand open, palm down. Imagine that your hand is the camera and the back of your hand represents the top of the camera. Now rotate your hand so that the "camera" is upside-down. You have just canted your hand around the roll axis. In cinematography, cant can be used to simulate various types of unsteady motion such as waves or turbulence, but can also be used for dramatic effect.</p>

<p>To accomplish this rotation around the Z axis using glMatrix:</p>

<pre class="brush: js">mat4.rotateZ(viewMatrix, viewMatrix, cantAngle);</pre>

<h2 id="Combining_movements">Combining movements</h2>

<p>You can perform multiple movements at once, such as zooming while panning, or tilting and canting at the same time.</p>

<h3 id="Translating_along_multiple_axes">Translating along multiple axes</h3>

<p>Translating along multiple axes is quite easy. Previously, we performed our translations like this:</p>

<pre class="brush: js">mat4.translate(viewMatrix, viewMatrix, [-truckDistance, 0, 0]);
mat4.translate(viewMatrix, viewMatrix, [0, -pedestalDistance, 0]);
mat4.translate(viewMatrix, viewMatrix, [0, 0, dollyDistance]);
</pre>

<p>The solution here is obvious. SInce the translation is expressed as a vector providing the distance to move along each axis, we can combine them like this:</p>

<pre class="brush: js">mat4.translate(viewMatrix, viewMatrix,
     [-truckDistance, -pedestalDistance, dollyDistance]);</pre>

<p>This will shift the origin of the matrix <code>viewMatrix</code> by the specified amount along each axis.</p>

<h3 id="Rotating_around_multiple_axes">Rotating around multiple axes</h3>

<p>You can also combine rotations around multiple axes into a single rotation around a quaternion representing a shared axis for the rotations. To perform the rotations separately, you use <a href="https://en.wikipedia.org/wiki/Euler_angles">Euler angles</a> (separate angles around each axis) to apply pitch, yaw, and roll like this:</p>

<pre class="brush: js">mat4.rotateX(viewMatrix, viewMatrix, pitchAngle);
mat4.rotateY(viewMatrix, viewMatrix, yawAngle);
mat4.rotateZ(viewMatrix, viewMatrix, rollAngle);
</pre>

<p>You can instead construct a {{Glossary("quaternion")}} representing a combined rotation axis from the Euler angles, then rotate the matrix using multiplication, like this:</p>

<pre class="brush: js">const axisQuat = quat.create();
const rotateMatrix = mat4.create();
quat.fromEuler(axisQuat, pitchAngle, yawAngle, rollAngle);
mat4.fromQuat(rotateMatrix, axisQuat);
mat4.multiply(viewMatrix, viewMatrix, rotateMatrix);</pre>

<p>This converts the Euler angles for pitch, yaw, and roll into a quaternion representing all three rotations. This is then converted into a rotation transform matrix; then, finally, the view matrix is multiplied by the rotation transform to complete the rotations.</p>

<h2 id="Representing_3D_with_WebXR">Representing 3D with WebXR</h2>

<p>WebXR takes 3D graphics a step further, allowing them to be presented using special visual hardware such as goggles or a headset to create 3D graphics that appear to actually exist in three dimensions, potentially within the context of the real world (in the case of augmented reality).</p>

<p>In order to perceive depth, it's necessary to have two perspectives on the scene. By comparing the two views, it's possible to recognize the depth of objects and, by extension, the distance between the viewer and objects which are seen. This is why we have two eyes, spaced slightly apart. You can remind yourself of this fact by closing one eye at a time, switching back and forth between the two eyes. Notice how your left eye can see the left side of your nose but not the right, while your right eye sees the right side of your nose but not the left. That's just one of many differences that exist between what each of your eyes see.</p>

<p>Our brain receives two sets of data about light levels and wavelengths throughout our field of view—one from each eye. The brain uses this data to construct the scene in our minds, using the slight differences between the two perspectives to figure out depth and distance.</p>

<h3 id="Rendering_the_scene">Rendering the scene</h3>

<p>An XR—shorthand that encompasses both virtual reality (VR) and augmented reality (AR)—headset presents 3D imagery to us by drawing two views of the scene, slightly offset from one another just like the views obtained by our two eyes. These views are then separately fed to each eye, in order to allow them to collect the data our brain needs in order to construct a 3D image in our minds.</p>

<p>To do this, WebXR asks your renderer to draw the scene twice for each frame of video—once for each eye. The two views are rendered into the same framebuffer, one on the left and one on the right. The XR device then uses screens and lenses to present the left half of the produced image to our left eye and the right half to our right eye.</p>

<p>For example, consider a device which uses a 2560x1440 pixel frame buffer. Dividing this into two parts—half for each eye—results in each eye's view being drawn at a resolution of 1280x1440 pixels. Here's what that looks like conceptually:</p>

<p><img alt="Diagram showing how a framebuffer is divided between two eyes' viewpoints" src="twoviewsoneframebuffer.svg"></p>

<p>Your code tells the WebXR engine that you want to provide the next animation frame by calling the {{domxref("XRSession")}} method {{domxref("XRSession.requestAnimationFrame", "requestAnimationFrame()")}}, providing a callback function that renders a frame of animation. When the browser needs you to render the scene, it invokes the callback, providing as input parameters the current time and an {{domxref("XRFrame")}} encapsulating the data needed to render the correct frame.</p>

<p>This information includes the {{domxref("XRViewerPose")}} describing the position and facing direction of the viewer within the scene as well as a list of {{domxref("XRView")}} objects, each representing one perspective on the scene. In current WebXR implementations, there will never be more than two entries in this list: one describing the position and viewing angle of the left eye and another doing the same for the right. You can tell which eye a given <code>XRView</code> represents by checking the value of its {{domxref("XRView.eye", "eye")}} property, which is a string whose value is <code>left</code> or <code>right</code> (a third possible value, <code>none</code>, theoretically may be used to represent another point of view, but support for this is not entirely available in the current API).</p>

<h3 id="Example_frame_callback">Example frame callback</h3>

<p>A fairly basic (but typical) callback for rendering frames might look like this:</p>

<pre class="brush: js">function myAnimationFrameCallback(time, frame) {
  let adjustedRefSpace = applyPositionOffsets(xrReferenceSpace);
  let pose = frame.getViewerPose(adjustedRefSpace);

  animationFrameRequestID = frame.session.requestAnimationFrame(myAnimationFrameCallback);

  if (pose) {
    let glLayer = frame.session.renderState.baseLayer;
    gl.bindFramebuffer(gl.FRAMEBUFFER, glLayer.framebuffer);
    CheckGLError("Binding the framebuffer");

    gl.clearColor(0, 0, 0, 1.0);
    gl.clearDepth(1.0);
    gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);
    CheckGLError("Clearing the framebuffer");

    const deltaTime = (time - lastFrameTime) * 0.001;
    lastFrameTime = time;

    for (let view of pose.views) {
      let viewport = glLayer.getViewport(view);
      gl.viewport(viewport.x, viewport.y, viewport.width, viewport.height);
      CheckGLError(`Setting viewport for eye: ${view.eye}`);

      myRenderScene(gl, view, sceneData, deltaTime);
    }
  }
}
</pre>

<p>The callback begins by calling a custom function, <code>applyPositionOffsets()</code>, which takes a reference space and applies to its transform matrix any changes that need to be made to take into account things such as user inputs from devices not controlled by WebXR, such as the keyboard and mouse. The adjusted {{domxref("XRReferenceSpace")}} returned by this function is then passed into the {{domxref("XRFrame")}} method {{domxref("XRFrame.getViewerPose", "getViewerPose()")}} to get the {{domxref("XRViewerPose")}} representing the viewer's position and viewing angle.</p>

<p>Next, we go ahead and queue up the request to render the next frame of video, so we don't have to worry about doing it later, by calling <code>requestAnimationFrame()</code> again.</p>

<p>Now it's time to render the scene. If we did successfully obtain a pose, we get the {{domxref("XRWebGLLayer")}} we need to use for rendering from the session's {{domxref("XRSession.renderState", "renderState")}} object's {{domxref("XRRenderState.baseLayer", "baseLayer")}} property. We bind this to WebGL's <code>gl.FRAMEBUFFER</code> target using the {{domxref("WebGLRenderingContext")}} method {{domxref("WebGLRenderingContext.bindFrameBuffer", "gl.bindFrameBuffer()")}}.</p>

<p>Then we clear the framebuffer to ensure we're starting with a known state, since our renderer will not be touching every pixel. We set the clear color to opaque black using {{domxref("WebGLRenderingContext.clearColor", "gl.clearColor()")}} and the value to clear the depth buffer to 1.0 by calling the {{domxref("WebGLRenderingContext")}} method {{domxref("WebGLRenderingContext.clearDepth", "gl.clearDepth()")}}. Then we call the {{domxref("WebGLRenderingContext")}} method {{domxref("WebGLRenderingContext.clear", "gl.clear()")}}, which clears the framebuffer (since we include <code>gl.COLOR_BUFFER_BIT</code> in the mask parameter) and the depth buffer (because we include <code>gl.DEPTH_BUFFER_BIT</code>).</p>

<p>Then we determine how much time has elapsed since the previous frame was rendered by comparing the frame's desired render time with the time at which the last frame was drawn. Since this value is in milliseconds, we convert it to seconds by multiplying by 0.001 (or dividing by 1000).</p>

<p>Now we loop over the pose's views, as found in the {{domxref("XRViewerPose")}} array, {{domxref("XRViewerPose.views", "views")}}. For each view, we ask the {{domxref("XRWebGLLayer")}} for the appropriate viewport to use, configure the WebGL viewport to match by passing the position and size information into {{domxref("WebGLRenderingContext.viewport", "gl.viewport()")}}. This constrains rendering so that we can only draw into the portion of the framebuffer that represents the image seen by the eye identified by {{domxref("XRView.eye", "view.eye")}}.</p>

<p>With the constraints so established and everything else we need ready, we call a custom function, <code>myRenderScene()</code>, to actually perform the computations and WebGL rendering to render the frame. In this case, we're passing in the WebGL context, <code>gl</code>, the {{domxref("XRView")}} <code>view</code>, a <code>sceneData</code> object (which contains things like the vertex and fragment shaders, vertex lists, textures, and so forth), and <code>deltaTime</code>, which indicates how much time has passed since the previous frame, so that we know how far to advance the animation.</p>

<p>When this function returns, the WebGL framebuffer being used by WebXR now has in it two copies of the scene, each occupying half the frame: one for the left eye, and one for the right eye. This makes its way through the XR software and drivers into the headset, where each half is shown to the appropriate eye.</p>

<h2 id="See_also">See also</h2>

<ul>
 <li><a href="/en-US/docs/Web/API/WebXR_Device_API/Geometry">Geometry and reference spaces</a></li>
 <li><a href="/en-US/docs/Web/API/WebGL_API/WebGL_model_view_projection">WebGL model view projection</a></li>
 <li><a href="/en-US/docs/Web/API/WebGL_API/Matrix_math_for_the_web">Matrix math for the web</a></li>
 <li><a href="/en-US/docs/Web/API/WebXR_Device_API/Movement_and_motion">Movement, orientation, and motion: A WebXR example</a></li>
</ul>