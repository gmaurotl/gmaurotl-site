<div class="researcher-profile">

  <header class="hero-section">
    <div class="hero-text">
      <h1>Mauro Gonzalo Tarazona Lévano</h1>
      <h2 class="subtitle">PhD Researcher in Wireless Communications & AI</h2>
      
      <p class="bio">
        Investigating the intersection of <span class="highlight">5G/6G systems</span> and <span class="highlight">Generative AI</span>. 
        Focusing on channel modeling and MIMO propagation.
      </p>

      <div class="tech-stack">
        <span class="badge">3GPP TR 38.901</span>
        <span class="badge">Sionna (Ray Tracing)</span>
        <span class="badge">Deep Learning</span>
      </div>
    </div>
    
    <div class="hero-image">
      <img src="/tu-foto-perfil.jpg" alt="Mauro Tarazona" class="avatar">
    </div>
  </header>

  <hr class="divider">

  <section>
    <h3>Research Interests</h3>
    <div class="grid-container">
      <div class="card">
        <div class="icon">📡</div>
        <h4>Channel Modeling</h4>
        <p>UMa / UMi, LoS / NLoS scenarios</p>
      </div>
      <div class="card">
        <div class="icon">📶</div>
        <h4>MIMO Systems</h4>
        <p>Spatial correlation & Massive MIMO</p>
      </div>
      <div class="card">
        <div class="icon">🧠</div>
        <h4>Generative AI</h4>
        <p>GANs & Diffusion models for Wireless</p>
      </div>
      <div class="card">
        <div class="icon">⚡</div>
        <h4>6G Propagation</h4>
        <p>Ray tracing & Measurement validation</p>
      </div>
    </div>
  </section>

  <section class="explore-section">
    <a href="/research/" class="btn-link">🔬 Research</a>
    <a href="/publications/" class="btn-link">📄 Publications</a>
    <a href="/blog/" class="btn-link">✍️ Blog</a>
  </section>

</div>

<style>
  /* CSS SUGERIDO (Pégalo en tu custom.css o bloque style) */
  
  /* Fuentes y Colores */
  .researcher-profile {
    font-family: 'Inter', system-ui, sans-serif;
    color: #333;
    max-width: 900px;
    margin: 0 auto;
    padding: 40px 20px;
  }

  /* Hero Layout */
  .hero-section {
    display: flex;
    align-items: center;
    gap: 40px;
    margin-bottom: 40px;
  }
  
  @media (max-width: 768px) {
    .hero-section { flex-direction: column-reverse; text-align: center; }
  }

  h1 { font-size: 2.2rem; margin-bottom: 0.5rem; letter-spacing: -0.5px; }
  .subtitle { font-size: 1.1rem; color: #666; font-weight: 400; margin-bottom: 1.5rem; }
  
  /* Foto */
  .avatar {
    width: 180px;
    height: 180px;
    border-radius: 50%; /* Circular */
    object-fit: cover;
    box-shadow: 0 10px 30px rgba(0,0,0,0.1);
  }

  /* Badges Tecnológicos */
  .tech-stack { margin-top: 15px; display: flex; gap: 10px; flex-wrap: wrap; }
  .badge {
    background: #f0f4f8;
    color: #2c3e50;
    padding: 4px 10px;
    border-radius: 4px;
    font-family: 'Fira Code', monospace; /* Toque técnico */
    font-size: 0.85rem;
    border: 1px solid #dae1e7;
  }

  /* Grid de Intereses */
  .grid-container {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 20px;
    margin-top: 20px;
  }

  .card {
    background: #fff;
    border: 1px solid #eaeaea;
    padding: 20px;
    border-radius: 8px;
    transition: transform 0.2s, box-shadow 0.2s;
  }
  
  .card:hover {
    transform: translateY(-3px);
    box-shadow: 0 5px 15px rgba(0,0,0,0.05);
    border-color: #007bff; /* Color de acento al pasar el mouse */
  }

  .icon { font-size: 1.5rem; margin-bottom: 10px; }
  .card h4 { margin: 0 0 5px 0; font-size: 1rem; }
  .card p { font-size: 0.85rem; color: #666; margin: 0; }

  /* Botones de Navegación */
  .explore-section {
    margin-top: 50px;
    display: flex;
    gap: 15px;
    justify-content: center;
  }

  .btn-link {
    text-decoration: none;
    padding: 10px 25px;
    border: 1px solid #ddd;
    border-radius: 50px;
    color: #333;
    font-weight: 500;
    transition: all 0.2s;
  }

  .btn-link:hover {
    background: #212529;
    color: #fff;
    border-color: #212529;
  }
  
  .divider { border: 0; border-top: 1px solid #eee; margin: 40px 0; }
</style>
