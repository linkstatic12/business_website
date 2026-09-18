(globalThis.TURBOPACK||(globalThis.TURBOPACK=[])).push(["object"==typeof document?document.currentScript:void 0,74030,e=>{"use strict";var t=e.i(43476),r=e.i(71645),o=e.i(46932),i=e.i(10542),l=e.i(95420),n=e.i(33768),a=e.i(72328);let u=`
// Simplex 2D noise
vec3 permute(vec3 x) { return mod(((x*34.0)+1.0)*x, 289.0); }
float snoise(vec2 v){
  const vec4 C = vec4(0.211324865405187, 0.366025403784439,
           -0.577350269189626, 0.024390243902439);
  vec2 i  = floor(v + dot(v, C.yy) );
  vec2 x0 = v -   i + dot(i, C.xx);
  vec2 i1;
  i1 = (x0.x > x0.y) ? vec2(1.0, 0.0) : vec2(0.0, 1.0);
  vec4 x12 = x0.xyxy + C.xxzz;
  x12.xy -= i1;
  i = mod(i, 289.0);
  vec3 p = permute( permute( i.y + vec3(0.0, i1.y, 1.0 ))
  + i.x + vec3(0.0, i1.x, 1.0 ));
  vec3 m = max(0.5 - vec3(dot(x0,x0), dot(x12.xy,x12.xy),
    dot(x12.zw,x12.zw)), 0.0);
  m = m*m ;
  m = m*m ;
  vec3 x = 2.0 * fract(p * C.www) - 1.0;
  vec3 h = abs(x) - 0.5;
  vec3 ox = floor(x + 0.5);
  vec3 a0 = x - ox;
  m *= 1.79284291400159 - 0.85373472095314 * ( a0*a0 + h*h );
  vec3 g;
  g.x  = a0.x  * x0.x  + h.x  * x0.y;
  g.yz = a0.yz * x12.xz + h.yz * x12.yw;
  return 130.0 * dot(m, g);
}

// Fbm layering for organic structure (Optimized: Reduced from 5 to 3 iterations for massive performance gain)
float fbm(vec2 x) {
    float v = 0.0;
    float a = 0.5;
    vec2 shift = vec2(100.0);
    // Rotate to reduce axial bias
    mat2 rot = mat2(cos(0.5), sin(0.5), -sin(0.5), cos(0.50));
    for (int i = 0; i < 3; ++i) { // DROPPED OFF 2 EXPENSIVE CYCLES
        v += a * snoise(x);
        x = rot * x * 2.0 + shift;
        a *= 0.5;
    }
    return v;
}

`,c=`
precision highp float;

uniform float uTime;
uniform vec2 uResolution;
uniform vec2 uMouse;
uniform float uScroll;
uniform float uVelocity;   // damped scroll speed, 0 at rest
uniform float uPointerVel; // damped cursor speed, 0 at rest

`,s=`
precision highp float;
uniform sampler2D uPrev;
uniform vec2 uPointer;     // field uv
uniform vec2 uPointerVel;  // field uv per second, scaled into -1..1
uniform float uInject;     // 1 while the pointer is moving, 0 otherwise
uniform float uDt;
uniform vec2 uTexel;
uniform float uAspect;

vec3 decode(vec4 c) { return vec3((c.rg - 0.5) * 2.0, c.b); }
vec4 encode(vec2 v, float ink) { return vec4(clamp(v * 0.5 + 0.5, 0.0, 1.0), clamp(ink, 0.0, 1.0), 1.0); }

void main() {
    vec2 uv = gl_FragCoord.xy * uTexel;
    vec3 here = decode(texture2D(uPrev, uv));
    // carried: read where this flow came from
    vec3 prev = decode(texture2D(uPrev, uv - here.xy * uDt * 0.9));
    // curled: neighbours give the local vorticity, the flow turns a little around it
    vec3 l = decode(texture2D(uPrev, uv - vec2(uTexel.x, 0.0)));
    vec3 r = decode(texture2D(uPrev, uv + vec2(uTexel.x, 0.0)));
    vec3 d = decode(texture2D(uPrev, uv - vec2(0.0, uTexel.y)));
    vec3 u = decode(texture2D(uPrev, uv + vec2(0.0, uTexel.y)));
    float curl = (r.y - l.y) - (u.x - d.x);
    vec2 vel = prev.xy + vec2(-prev.y, prev.x) * curl * 3.0 * uDt;
    float ink = prev.z;
    // settling
    vel *= exp(-uDt * 0.9);
    ink *= exp(-uDt * 0.7);
    // the pointer writes a round splat carrying its velocity, and stirs in proportion to it
    vec2 dpx = (uv - uPointer) * vec2(uAspect, 1.0);
    float g = exp(-dot(dpx, dpx) / 0.0028);
    vel += uPointerVel * g * uInject;
    ink += length(uPointerVel) * 0.7 * uInject * g;
    if (length(vel) < 0.006) vel = vec2(0.0);
    if (ink < 0.004) ink = 0.0;
    gl_FragColor = encode(vel, ink);
}
`,f={classic:c+u+`
void main() {
    // Coordinates mapping
    vec2 uv = gl_FragCoord.xy / uResolution.xy;
    vec2 p = uv * 2.0 - 1.0;
    p.x *= uResolution.x / uResolution.y;

    // MOUSE INFLUENCE (The Fluid Reacts to Cursor)
    // uMouse is normalized screen space but center is (0,0). uv is (0,0) bottom-left.
    // We adjust uMouse mapping slightly to match p which goes -1 to 1.
    vec2 mouseP = uMouse;

    // We create a pulling effect: the fluid distorts towards the mouse.
    float dist = length(p - mouseP);
    // Smooth, large area of effect. A fast cursor bites deeper than a drifting one, so the
    // surface reads as something with weight rather than a static falloff following the pointer.
    float mousePull = exp(-dist * 1.5) * (0.5 + uPointerVel * 0.55);

    // Core fluid calculation using fbm and time
    float t = uTime * 0.15;

    // Domain warp driven by mouse and time
    vec2 q = vec2(0.);
    q.x = fbm( p + 0.00*t );
    q.y = fbm( p + vec2(1.0) );

    vec2 r = vec2(0.);
    // Inject the mousePull into the second layer of distortion to make the fluid flow toward the cursor.
    // Scrolling no longer drags the liquid: the drag read as the surface stalling and turning with
    // the scroll, a lag that was not there. The speed only lifts the light a little.
    // and it settles again the moment you stop.
    r.x = fbm( p + 1.0*q + vec2(1.7,9.2) + 0.15*t + (mouseP.x * mousePull) );
    r.y = fbm( p + 1.0*q + vec2(8.3,2.8) + 0.126*t + (mouseP.y * mousePull) );

    // Final turbulence
    float f = fbm(p + r + (uScroll * 0.2));

    // Color mapping - Dark, silvery, high-end "liquid data"
    vec3 colorBase = vec3(0.02, 0.02, 0.03);
    vec3 colorMid = vec3(0.08, 0.12, 0.15);
    vec3 colorHigh = vec3(0.4, 0.5, 0.6);

    // Map f to a smooth gradient, boosted slightly around the mouse
    vec3 color = mix(colorBase, colorMid, clamp((f*f)*4.0, 0.0, 1.0));
    color = mix(color, colorHigh, clamp(length(q)*length(r)*f, 0.0, 1.0) * (0.6 + mousePull * 0.4));

    // REMOVED inner vignette so the fluid covers the entire 100vw/100vh canvas.
    // Instead of completely blacking out the edges, we just let it fade very softly
    float v = smoothstep(2.5, 0.1, length(p));

    // A subtle iridescent edge line (mimics silk folds catching light)
    float edge = smoothstep(0.4, 0.5, f) - smoothstep(0.5, 0.6, f);
    // Silk catches more light while it is moving.
    color += vec3(0.2, 0.3, 0.4) * edge * (0.5 + uVelocity * 0.1);

    // Apply a very light multiplier to keep it visible everywhere
    color *= (0.4 + 0.6 * v);

    // Fade to pure black at the bottom for a seamless scroll transition
    float bottomFade = smoothstep(0.0, 0.2, uv.y);
    color *= bottomFade;

    gl_FragColor = vec4(color, 1.0);
}
`,next:c+u+`
uniform sampler2D uField;  // the wake: flow in rg (0.5 = still), stir in b

vec3 readField(vec2 at) {
    vec4 c = texture2D(uField, at);
    return vec3((c.rg - 0.5) * 2.0, c.b);
}

void main() {
    vec2 uv = gl_FragCoord.xy / uResolution.xy;
    vec2 p = uv * 2.0 - 1.0;
    p.x *= uResolution.x / uResolution.y;

    // the pull follows a weighted bead, not the cursor itself, so a fast move stretches the silk
    vec2 mouseP = uMouse;
    float dist = length(p - mouseP);
    float mousePull = exp(-dist * 1.5) * (0.5 + uPointerVel * 0.55);
    float t = uTime * 0.15;

    // the wake you left behind: the silk is carried along the flow and stirred where you passed.
    // Nothing is painted onto the surface; both only move the field the silk is drawn from.
    vec3 field = readField(uv);
    vec2 flow = field.xy;
    float stir = field.z;

    vec2 q = vec2(fbm(p), fbm(p + vec2(1.0)));
    vec2 r;
    r.x = fbm(p + q + vec2(1.7, 9.2) + 0.15 * t + (mouseP.x * mousePull));
    r.y = fbm(p + q + vec2(8.3, 2.8) + 0.126 * t + (mouseP.y * mousePull));
    // the scroll does not drag the silk (it read as a stall and a turn, a lag that was not there)
    vec2 w = p + r + (uScroll * 0.2) + flow * 0.75 + stir * 0.4 * vec2(r.y, -r.x);
    float f = fbm(w);

    // relief: the slope of the last octave set, the warp held still for it
    float e = 0.07;
    float fx = fbm(w + vec2(e, 0.0));
    float fy = fbm(w + vec2(0.0, e));
    vec3 n = normalize(vec3(-(fx - f) / e * 0.16, -(fy - f) / e * 0.16, 1.0));

    // the light: a slow orbit, drawn a little toward the cursor
    vec3 L = normalize(vec3(0.55 * cos(uTime * 0.11) + mouseP.x * 0.25, 0.45 * sin(uTime * 0.09) + mouseP.y * 0.25 + 0.35, 0.85));
    vec3 V = vec3(0.0, 0.0, 1.0);
    vec3 H = normalize(L + V);
    float ndl = max(dot(n, L), 0.0);
    float ndh = max(dot(n, H), 0.0);
    float ndv = max(dot(n, V), 0.0);
    float sheen = pow(ndh, 12.0) * 0.12;
    float glint = pow(ndh, 72.0) * 0.32;
    float fresnel = pow(1.0 - ndv, 3.2);
    // the light lives on the folds' crests, where the classic surface already caught it
    float crest = clamp(length(q) * length(r) * f * 1.6, 0.0, 1.0);

    vec3 colorBase = vec3(0.02, 0.02, 0.03);
    vec3 colorMid = vec3(0.08, 0.12, 0.15);
    vec3 colorHigh = vec3(0.4, 0.5, 0.6);
    vec3 albedo = mix(colorBase, colorMid, clamp((f * f) * 4.0, 0.0, 1.0));
    albedo = mix(albedo, colorHigh, clamp(length(q) * length(r) * f, 0.0, 1.0) * (0.6 + mousePull * 0.4));
    vec3 color = albedo * (0.78 + 0.22 * ndl);

    // thin film: the optical path grows with the fold's height and the viewing angle
    float path = (f * 0.5 + 0.5) * 2.2 + (1.0 - ndv) * 1.6;
    vec3 film = 0.5 + 0.5 * cos(6.28318 * (path + vec3(0.0, 0.33, 0.67)));
    film = mix(vec3(dot(film, vec3(0.3333))), film, 0.4);
    film = mix(film, colorHigh * 1.6, 0.5);
    color += film * (fresnel * 0.16 + sheen * 1.0 + glint * 0.7) * (0.45 + 0.55 * crest) * (0.75 + uVelocity * 0.08);
    color += colorHigh * glint * 0.45 * crest;

    // the silk fold lines, kept from the classic surface
    float edge = smoothstep(0.4, 0.5, f) - smoothstep(0.5, 0.6, f);
    color += vec3(0.2, 0.3, 0.4) * edge * (0.35 + uVelocity * 0.1);

    float v = smoothstep(2.5, 0.1, length(p));
    color *= (0.4 + 0.6 * v);

    // grade: a slight lift and a gentle curve; then the bottom fade to pure black for the scroll
    color = pow(max(color, 0.0), vec3(0.97)) * 0.99 + 0.004;
    float bottomFade = smoothstep(0.0, 0.2, uv.y);
    color *= bottomFade;

    // dither so the dark gradients never band
    float ign = fract(52.9829189 * fract(0.06711056 * gl_FragCoord.x + 0.00583715 * gl_FragCoord.y));
    color += (ign - 0.5) / 255.0;

    gl_FragColor = vec4(color, 1.0);
}
`},m=`
attribute vec2 position;

void main() {
    gl_Position = vec4(position, 0.0, 1.0);
}
`,d=["uTime","uResolution","uMouse","uScroll","uVelocity","uPointerVel","uField"],h=["uPrev","uPointer","uPointerVel","uInject","uDt","uTexel","uAspect"];function v(e,t,r){let o=e.createShader(t);return o?(e.shaderSource(o,r),e.compileShader(o),e.getShaderParameter(o,e.COMPILE_STATUS))?o:(console.error(e.getShaderInfoLog(o)),e.deleteShader(o),null):null}function x(e,t,r){let o=v(e,e.VERTEX_SHADER,m),i=v(e,e.FRAGMENT_SHADER,t),l=e.createProgram();if(!o||!i||!l)return r(null);e.attachShader(l,o),e.attachShader(l,i),e.linkProgram(l);let n=e.getExtension("KHR_parallel_shader_compile"),a=()=>{var t,o;let i,n,a;if(!e.getProgramParameter(l,e.LINK_STATUS))return console.error(e.getProgramInfoLog(l)),r(null);r((t=e,o=l,t.useProgram(o),i=t.createBuffer(),t.bindBuffer(t.ARRAY_BUFFER,i),t.bufferData(t.ARRAY_BUFFER,new Float32Array([-1,-1,1,-1,-1,1,1,1]),t.STATIC_DRAW),n=t.getAttribLocation(o,"position"),t.enableVertexAttribArray(n),t.vertexAttribPointer(n,2,t.FLOAT,!1,0,0),a=Object.fromEntries(d.map(e=>[e,t.getUniformLocation(o,e)])),{program:o,uniforms:a,position:n,buffer:i}))};if(!n)return a();let u=()=>{e.isContextLost()||(e.getProgramParameter(l,n.COMPLETION_STATUS_KHR)?a():requestAnimationFrame(u))};u()}function g({still:e=!1,variant:o="next"}){let i=(0,r.useRef)(null),l=(0,r.useRef)(e),n=(0,r.useRef)(()=>{});return(0,r.useEffect)(()=>{let e=i.current;if(!e)return;let t=e.getContext("webgl",{antialias:!1,alpha:!1,depth:!1,stencil:!1,powerPreference:"high-performance"});if(!t)return;let r=null;t.isContextLost()&&t.getExtension("WEBGL_lose_context")?.restoreContext();let a={x:0,y:0},u={x:0,y:0},c=0,d=0,g=0,p=0,w=window.scrollY,E=0,y="next"===o,b=null,T={x:.5,y:.5},R={x:0,y:0},P=.5,F=.5,A=0,M=!1,_={x:0,y:0,vx:0,vy:0},L=e=>{let t=e.clientX/window.innerWidth*2-1,r=-(e.clientY/window.innerHeight*2-1);if(u.x=t*window.innerWidth/window.innerHeight,u.y=r,c=Math.min(c+6*Math.hypot(t-g,r-p),1),g=t,p=r,y){let t=performance.now(),r=e.clientX/window.innerWidth,o=1-e.clientY/window.innerHeight,i=Math.max(.004,(t-A)/1e3);R.x=Math.max(-1,Math.min(1,(r-P)/i*.3)),R.y=Math.max(-1,Math.min(1,(o-F)/i*.3)),T.x=r,T.y=o,P=r,F=o,A=t,M=!0}},S=()=>{let e=window.scrollY;d=Math.min(d+Math.abs(e-w)/220,1),w=e};window.addEventListener("pointermove",L,{passive:!0}),window.addEventListener("scroll",S,{passive:!0});let C=0,D=0,U=0,k=()=>{D=Math.max(1,Math.round(e.clientWidth)),U=Math.max(1,Math.round(e.clientHeight)),I()},I=()=>{if(!D)return;C||(C=Math.min(1,Math.sqrt(16e5/(D*U))));let r=Math.max(1,Math.round(D*C)),o=Math.max(1,Math.round(U*C));(e.width!==r||e.height!==o)&&(e.width=r,e.height=o,t.viewport(0,0,r,o),e.dataset.scale=C.toFixed(2))};k(),window.addEventListener("resize",k);let B=1e3/60,V=0,O=0,z=0,N=[],H=performance.now(),X=H,q=0,j=o=>{q=0,(o=>{if(!r)return;let i=Math.min(.05,(o-X)/1e3);l.current||((e,t)=>{if(z||(z=e),t<3||t>250||(N.push(t),N.length<40))return;let r=N.slice().sort((e,t)=>e-t);N.length=0;let o=r[Math.floor(.1*r.length)],i=r[r.length>>1];o<.9*B?(V&&Math.abs(V-o)<.1*o&&(B=Math.max(4,o)),V=o):V=0,!(e-z<1500)&&(i>1.35*B?(O=0,C>.4&&(C=Math.max(.4,.85*C),I())):i<1.1*B&&C<1?(O+=1)>=5&&(O=0,C=Math.min(1,1.08*C),I()):O=0)})(o,o-X),X=o,I(),a.x+=(u.x-a.x)*.05,a.y+=(u.y-a.y)*.05;let n=Math.exp(-(2.2*i));if(d*=n,c*=n,E+=.001,y){let n=(u.x-_.x)*70-9*_.vx,a=(u.y-_.y)*70-9*_.vy;if(_.vx+=n*i,_.vy+=a*i,_.x+=_.vx*i,_.y+=_.vy*i,b&&!l.current){let l=M&&o-A<90?1:0;M=!1;let n=1^b.current;t.useProgram(b.program),t.bindBuffer(t.ARRAY_BUFFER,r.buffer),t.enableVertexAttribArray(b.position),t.vertexAttribPointer(b.position,2,t.FLOAT,!1,0,0),t.bindFramebuffer(t.FRAMEBUFFER,b.framebuffers[n]),t.viewport(0,0,256,256),t.activeTexture(t.TEXTURE0),t.bindTexture(t.TEXTURE_2D,b.textures[b.current]);let a=b.uniforms;t.uniform1i(a.uPrev,0),t.uniform2f(a.uPointer,T.x,T.y),t.uniform2f(a.uPointerVel,R.x,R.y),t.uniform1f(a.uInject,l),t.uniform1f(a.uDt,i),t.uniform2f(a.uTexel,.00390625,.00390625),t.uniform1f(a.uAspect,e.width/e.height),t.drawArrays(t.TRIANGLE_STRIP,0,4),b.current=n,t.bindFramebuffer(t.FRAMEBUFFER,null),t.viewport(0,0,e.width,e.height),t.useProgram(r.program),t.bindBuffer(t.ARRAY_BUFFER,r.buffer),t.enableVertexAttribArray(r.position),t.vertexAttribPointer(r.position,2,t.FLOAT,!1,0,0)}}let s=r.uniforms;t.uniform1f(s.uTime,(o-H)/1e3),t.uniform2f(s.uResolution,e.width,e.height),t.uniform2f(s.uMouse,y?_.x:a.x,y?_.y:a.y),t.uniform1f(s.uScroll,E),t.uniform1f(s.uVelocity,d),t.uniform1f(s.uPointerVel,c),y&&(t.activeTexture(t.TEXTURE0),t.bindTexture(t.TEXTURE_2D,b?b.textures[b.current]:null),t.uniform1i(s.uField,0)),t.drawArrays(t.TRIANGLE_STRIP,0,4)})(o),l.current||(q=requestAnimationFrame(j))};n.current=()=>{!q&&r&&(q=requestAnimationFrame(j))};let G=e=>{r=e,y&&e&&e.buffer&&(b=function(e,t){let r=v(e,e.VERTEX_SHADER,m),o=v(e,e.FRAGMENT_SHADER,s),i=e.createProgram();if(!r||!o||!i)return null;if(e.attachShader(i,r),e.attachShader(i,o),e.linkProgram(i),!e.getProgramParameter(i,e.LINK_STATUS))return console.error(e.getProgramInfoLog(i)),null;let l=[],n=[];for(let t=0;t<2;t+=1){let t=e.createTexture(),r=e.createFramebuffer();if(!t||!r)return null;e.bindTexture(e.TEXTURE_2D,t),e.texImage2D(e.TEXTURE_2D,0,e.RGBA,256,256,0,e.RGBA,e.UNSIGNED_BYTE,null),e.texParameteri(e.TEXTURE_2D,e.TEXTURE_MIN_FILTER,e.LINEAR),e.texParameteri(e.TEXTURE_2D,e.TEXTURE_MAG_FILTER,e.LINEAR),e.texParameteri(e.TEXTURE_2D,e.TEXTURE_WRAP_S,e.CLAMP_TO_EDGE),e.texParameteri(e.TEXTURE_2D,e.TEXTURE_WRAP_T,e.CLAMP_TO_EDGE),e.bindFramebuffer(e.FRAMEBUFFER,r),e.framebufferTexture2D(e.FRAMEBUFFER,e.COLOR_ATTACHMENT0,e.TEXTURE_2D,t,0),e.viewport(0,0,256,256),e.clearColor(.5,.5,0,1),e.clear(e.COLOR_BUFFER_BIT),l.push(t),n.push(r)}e.bindFramebuffer(e.FRAMEBUFFER,null);let a=Object.fromEntries(h.map(t=>[t,e.getUniformLocation(i,t)]));return{textures:[l[0],l[1]],framebuffers:[n[0],n[1]],program:i,uniforms:a,position:e.getAttribLocation(i,"position"),current:0}}(t,e.buffer)),e&&(t.useProgram(e.program),t.bindBuffer(t.ARRAY_BUFFER,e.buffer),t.enableVertexAttribArray(e.position),t.vertexAttribPointer(e.position,2,t.FLOAT,!1,0,0)),n.current()};t.isContextLost()||x(t,f[o],G);let Y=e=>{e.preventDefault(),q&&cancelAnimationFrame(q),q=0,r=null},W=()=>{x(t,f[o],e=>{k(),G(e)})};return e.addEventListener("webglcontextlost",Y),e.addEventListener("webglcontextrestored",W),()=>{q&&cancelAnimationFrame(q),window.removeEventListener("pointermove",L),window.removeEventListener("scroll",S),b&&(t.deleteProgram(b.program),b.textures.forEach(e=>t.deleteTexture(e)),b.framebuffers.forEach(e=>t.deleteFramebuffer(e)),b=null),window.removeEventListener("resize",k),e.removeEventListener("webglcontextlost",Y),e.removeEventListener("webglcontextrestored",W),r&&t.deleteProgram(r.program),r=null}},[o]),(0,r.useEffect)(()=>{l.current=e,n.current()},[e]),(0,t.jsx)("div",{className:"absolute inset-0 z-0",style:{width:"100vw",height:"100vh",left:0,top:0,overflow:"hidden"},children:(0,t.jsx)("canvas",{ref:i,"data-variant":o,style:{display:"block",background:"#000",width:"100%",height:"100%"}})})}function p(){let{scrollY:e}=(0,i.useScroll)(),u=(0,a.useReducedMotion)(),[c,s]=(0,r.useState)(!0),[f,m]=(0,r.useState)("next");(0,r.useEffect)(()=>{"classic"===new URLSearchParams(window.location.search).get("fluid")&&m("classic")},[]);let d=(0,r.useRef)(900);(0,r.useEffect)(()=>{let e=()=>{d.current=window.innerHeight};return e(),window.addEventListener("resize",e),()=>window.removeEventListener("resize",e)},[]);let h=()=>{let e=document.querySelector("footer");if(!e)return 0;let t=e.getBoundingClientRect().top;return Math.min(1,Math.max(0,(d.current-t)/(.7*d.current)))},v=(0,l.useTransform)(e,e=>{let t=1.9*d.current;return Math.max(Math.min(1,Math.max(0,1-(e-t)/(2.9*d.current-t))),h())});return(0,n.useMotionValueEvent)(e,"change",e=>{let t=e<2.9*d.current||h()>0;s(e=>e===t?e:t)}),(0,t.jsxs)(o.motion.div,{"aria-hidden":"true",style:{opacity:u?1:v},className:"fixed inset-0 z-0",children:[(0,t.jsx)(g,{still:!!u||!c,variant:f}),(0,t.jsx)("div",{className:"absolute inset-0 w-full h-full opacity-[0.02] mix-blend-screen pointer-events-none z-0",style:{backgroundImage:"url(\"data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noiseFilter'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.8' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noiseFilter)'/%3E%3C/svg%3E\")"}})]})}e.s(["default",()=>p],74030)},888,e=>{e.n(e.i(74030))}]);