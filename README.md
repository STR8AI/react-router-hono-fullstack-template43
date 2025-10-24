<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>LUKAIRO | Delivering Control. Returning Time.</title>

  <style>
    html, body {
      margin: 0;
      padding: 0;
      background: #000;
      color: #fff;
      font-family: 'Inter','SF Pro Display',-apple-system,BlinkMacSystemFont,sans-serif;
      overflow: hidden;
    }
    #lukairo-scene {
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100vh;
      z-index: 1;
      background: radial-gradient(circle at center,#040404,#000);
    }
    .overlay {
      position: fixed;
      top: 50%; left: 50%;
      transform: translate(-50%,-50%);
      text-align: center;
      color: #00E5D1;
      z-index: 5;
    }
    .overlay h1 {
      font-size: clamp(3rem,6vw,5rem);
      margin-bottom: 1rem;
      text-shadow: 0 0 40px rgba(0,229,209,0.4);
      letter-spacing: .05em;
    }
    .overlay p {
      color: #b0b0b0;
      font-size: 1.2rem;
      line-height: 1.6;
    }
    .maple{font-size:1.3rem;margin-left:.3rem;}
    footer{
      position:fixed;
      bottom:1rem;left:50%;
      transform:translateX(-50%);
      color:#444;font-size:.9rem;
      z-index:10;
    }
  </style>
</head>
<body>

  <div id="lukairo-scene"></div>

  <div class="overlay">
    <h1>LUKAIRO</h1>
    <p>Delivering Control. Returning Time.<br>Proudly Canadian <span class="maple">🍁</span></p>
  </div>

  <footer>© 2025 LUKAIRO LDB INC.</footer>

  <script src="https://unpkg.com/three@0.158.0/build/three.min.js"></script>

  <script>
  window.addEventListener("load", () => {
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(65, window.innerWidth / window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer({antialias:true,alpha:true});
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(window.devicePixelRatio);
    document.getElementById("lukairo-scene").appendChild(renderer.domElement);

    // --- LIGHTS ---
    const ambient = new THREE.AmbientLight(0x00e5d1,0.4);
    scene.add(ambient);
    const mainLight = new THREE.PointLight(0x00e5d1,1.6,100);
    scene.add(mainLight);
    const rotatingLight = new THREE.PointLight(0x00ffff,1,50);
    scene.add(rotatingLight);

    // --- CORE SPHERE (Gear texture) ---
    const loader = new THREE.TextureLoader();
    const texture = loader.load("LUKAIRO.png");
    const core = new THREE.Mesh(
      new THREE.SphereGeometry(3,64,64),
      new THREE.MeshStandardMaterial({
        map:texture,
        metalness:1,
        roughness:0.25,
        emissive:0x001111,
        emissiveIntensity:0.6
      })
    );
    scene.add(core);

    // --- CONNECTED PLATFORMS (Stars) ---
    const systems = [
      {name:"Jobber", orbit:8, color:0x00e5d1},
      {name:"Housecall Pro", orbit:10, color:0x00bfff},
      {name:"ServiceTitan", orbit:12, color:0x0099ff},
      {name:"Workiz", orbit:9, color:0x33ffee},
      {name:"Others", orbit:11, color:0x00ffcc}
    ];
    const nodes=[];
    const fontCanvas=document.createElement("canvas");
    const ctx=fontCanvas.getContext("2d");

    systems.forEach((sys,i)=>{
      const mat=new THREE.MeshStandardMaterial({
        emissive:sys.color,emissiveIntensity:2,
        color:0x001010,metalness:0.9,roughness:0.2
      });
      const orb=new THREE.Mesh(new THREE.SphereGeometry(0.4,32,32),mat);
      orb.userData={orbit:sys.orbit,phase:i,scale:1,dir:1,name:sys.name};
      scene.add(orb);
      nodes.push(orb);
    });

    // --- PARTICLE FIELD ---
    const geo=new THREE.BufferGeometry();
    const pos=new Float32Array(2500);
    for(let i=0;i<pos.length;i++) pos[i]=(Math.random()-0.5)*100;
    geo.setAttribute("position",new THREE.BufferAttribute(pos,3));
    const particles=new THREE.Points(
      geo,
      new THREE.PointsMaterial({color:0x00e5d1,size:0.05,transparent:true,opacity:0.4})
    );
    scene.add(particles);

    camera.position.z=22;

    // --- ANIMATION LOOP ---
    function animate(){
      requestAnimationFrame(animate);
      const t=Date.now()*0.001;

      // Core rotation
      core.rotation.y+=0.0025;
      core.rotation.x+=0.0012;

      // Rotating shimmer light
      rotatingLight.position.set(Math.sin(t*0.8)*10,Math.cos(t*0.6)*5,Math.cos(t*0.9)*10);

      // Particle slow drift
      particles.rotation.y+=0.0004;

      // Orbiting platforms
      nodes.forEach((n,i)=>{
        const angle=t*0.6+i;
        const pull=Math.sin(t*0.4+i)*0.3; // attraction oscillation
        const r=n.userData.orbit - pull;
        n.position.x=Math.sin(angle)*r;
        n.position.y=Math.cos(angle)*r;
        n.position.z=Math.sin(t*0.2+i)*1.5;
        // Pulse scale
        n.userData.scale+=0.002*n.userData.dir;
        if(n.userData.scale>1.1||n.userData.scale<0.9)n.userData.dir*=-1;
        n.scale.setScalar(n.userData.scale);
      });

      renderer.render(scene,camera);
    }
    animate();

    // --- RESIZE ---
    window.addEventListener("resize",()=>{
      camera.aspect=window.innerWidth/window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth,window.innerHeight);
    });
  });
  </script>
</body>
</html>