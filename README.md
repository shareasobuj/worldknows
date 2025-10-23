 worldknows
<html lang="bn">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>worldknows</title>
  <!-- Bootstrap CSS -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
  <style>
    /* Custom styles */
    body { background: fuchsia; font-family: Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial; }
    .logo { font-weight: 800; font-size: 26px; letter-spacing: 0.6px; color: #0d6efd; }
    .hero { background: linear-gradient(120deg, #e9f2ff 0%, #ffffff 100%); padding: 60px 0; }
    .card-hover:hover { transform: translateY(-6px); box-shadow: 0 10px 30px rgba(13,110,253,0.08); }
    .nav-link.active { font-weight: 600; }
    footer { background:#0d6efd; color: white; padding: 28px 0; }
    .page-title { margin-bottom: 24px; }
    .editor { min-height:120px; }
  </style>
</head>
<body>
  <div id="root"></div>

  <!-- React & Babel (for single-file demo) -->
  <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>

  <script type="text/babel">
    const { useState, useEffect } = React;

    /*************** LocalStorage helpers ***************/
    const LS = {
      get(key, fallback) { try { const v = localStorage.getItem(key); return v ? JSON.parse(v) : fallback; } catch(e){return fallback} },
      set(key, val){ localStorage.setItem(key, JSON.stringify(val)); }
    };

    // Default content for pages (will be saved to localStorage on first run)
    const defaultPages = {
      Home: {
        title: 'Welcome to worldknows',
        body: 'একটি নমুনা ১০-পেইজ রিএক্ট+বুটস্ট্র্যাপ ওয়েবসাইট। এখানে আপনি কনটেন্ট পরিবর্তন করে নিজস্ব ওয়েবসাইট বানাতে পারবেন।'
      },
      About: { title: 'About worldknows', body: 'worldknows একটি ডেমো ওয়েবসাইট। আপনি এখানে আপনার প্রতিষ্ঠানের মিশন, ভিশন, ইতিহাস ইত্যাদি বর্ণনা করতে পারবেন।' },
      Services: { title: 'Services', body: 'আমাদের সার্ভিসগুলো: কনসাল্টিং, ডেভেলপমেন্ট, ট্রেনিং।' },
      Blog: { title: 'Blog', body: 'এখানে ব্লগ পোস্টগুলো দেখানো হবে।' },
      Gallery: { title: 'Gallery', body: 'ইমেজ গ্যালারি।' },
      Team: { title: 'Team', body: 'টিম মেম্বারদের প্রোফাইল।' },
      Resources: { title: 'Resources', body: 'ডকুমেন্টেশন ও টিউটোরিয়াল।' },
      FAQ: { title: 'FAQ', body: 'সাধারণ জিজ্ঞাসা এবং উত্তর।' },
      Contact: { title: 'Contact', body: 'যোগাযোগ ফর্ম এখানে থাকবে।' },
      Dashboard: { title: 'Dashboard', body: 'অ্যাডমিন ড্যাশবোর্ড' }
    };

    /*************** Initial setup on first load ***************/
    function initIfNeeded(){
      // pages
      if(!localStorage.getItem('wk_pages')){
        LS.set('wk_pages', defaultPages);
      }
      // users: create default admin user if not present
      if(!localStorage.getItem('wk_users')){
        const users = [
          { id: 'admin', password: 'admin123', name: 'Admin', role: 'admin' }
        ];
        LS.set('wk_users', users);
      }
      // visitors
      if(!localStorage.getItem('wk_visitors')){
        LS.set('wk_visitors', []);
      }
      // track visitor info on page load
      recordVisitor();
    }

    function recordVisitor(){
      try{
        const visitors = LS.get('wk_visitors', []);
        const now = new Date().toISOString();
        const info = { time: now, ua: navigator.userAgent, language: navigator.language || null };
        visitors.push(info);
        // keep only last 200
        if(visitors.length>200) visitors.splice(0, visitors.length-200);
        LS.set('wk_visitors', visitors);
      }catch(e){console.warn('visitor record failed', e)}
    }

    /*************** App Component ***************/
    function App(){
      initIfNeeded();
      const pages = Object.keys(LS.get('wk_pages', defaultPages));
      const [page, setPage] = useState('Home');
      const [currentUser, setCurrentUser] = useState(LS.get('wk_currentUser', null));

      useEffect(()=>{
        LS.set('wk_currentUser', currentUser);
      },[currentUser]);

      return (
        <div>
          <Header current={page} onNav={(p)=>setPage(p)} pages={pages} currentUser={currentUser} onLogin={()=>setPage('Dashboard')} onLogout={()=>setCurrentUser(null)} />

          <main className="container py-5">
            <PageView page={page} onNavigate={setPage} currentUser={currentUser} setCurrentUser={setCurrentUser} />
          </main>

          <Footer />
        </div>
      )
    }

    /*************** Header ***************/
    function Header({current,onNav,pages,currentUser,onLogin,onLogout}){
      return (
        <nav className="navbar navbar-expand-lg navbar-light bg-white shadow-sm">
          <div className="container">
            <a className="navbar-brand d-flex align-items-center" href="#" onClick={(e)=>{e.preventDefault(); onNav('Home')}}>
              <div className="logo">worldknows</div>
            </a>
            <button className="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navMenu">
              <span className="navbar-toggler-icon"></span>
            </button>
            <div className="collapse navbar-collapse" id="navMenu">
              <ul className="navbar-nav ms-auto">
                {pages.map(p=> (
                  <li className="nav-item" key={p}>
                    <a href="#" className={"nav-link "+(current===p? 'active' : '')} onClick={(e)=>{e.preventDefault(); onNav(p)}}>{p}</a>
                  </li>
                ))}
                {!currentUser && (
                  <li className="nav-item">
                    <a href="#" className="nav-link" onClick={(e)=>{e.preventDefault(); onNav('Login')}}>Login</a>
                  </li>
                )}
                {currentUser && (
                  <li className="nav-item dropdown">
                    <a className="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">{currentUser.name}</a>
                    <ul className="dropdown-menu dropdown-menu-end">
                      <li><a className="dropdown-item" href="#" onClick={(e)=>{e.preventDefault(); onNav('Dashboard')}}>Dashboard</a></li>
                      <li><a className="dropdown-item" href="#" onClick={(e)=>{e.preventDefault(); onLogout()}}>Logout</a></li>
                    </ul>
                  </li>
                )}
              </ul>
            </div>
          </div>
        </nav>
      )
    }

    /*************** Page router ***************/
    function PageView({page,onNavigate,currentUser,setCurrentUser}){
      switch(page){
        case 'Home': return <Home onNavigate={onNavigate} />;
        case 'About': return <About />;
        case 'Services': return <Services />;
        case 'Blog': return <Blog />;
        case 'Gallery': return <Gallery />;
        case 'Team': return <Team />;
        case 'Resources': return <Resources />;
        case 'FAQ': return <FAQ />;
        case 'Contact': return <Contact />;
        case 'Dashboard': return <Dashboard currentUser={currentUser} />;
        case 'Login': return <Auth setCurrentUser={setCurrentUser} onSuccess={()=>onNavigate('Dashboard')} />;
        default: return <Home onNavigate={onNavigate} />;
      }
    }

    /*************** Pages (read content from localStorage) ***************/
    function usePageContent(name){
      const [pages,setPages] = useState(LS.get('wk_pages', defaultPages));
      useEffect(()=>{ LS.set('wk_pages', pages); }, [pages]);
      return [pages[name], (newContent)=>{ setPages(prev=>({ ...prev, [name]: { ...prev[name], ...newContent } })) }];
    }

    function Home({onNavigate}){
      const [content] = usePageContent('Home');
      return (
        <div>
          <section className="hero rounded-3 mb-4">
            <div className="container">
              <div className="row align-items-center">
                <div className="col-md-7">
                  <h1 className="display-6">{content.title}</h1>
                  <p className="lead">{content.body}</p>
                  <div className="mt-4">
                    <button className="btn btn-primary me-2" onClick={()=>onNavigate('Services')}>Our Services</button>
                    <button className="btn btn-outline-primary" onClick={()=>onNavigate('Contact')}>Contact Us</button>
                  </div>
                </div>
                <div className="col-md-5 text-center">
                  <img src="https://images.unsplash.com/photo-1507525428034-b723cf961d3e?q=80&w=800&auto=format&fit=crop&ixlib=rb-4.0.3&s=placeholder" alt="world" className="img-fluid rounded" />
                </div>
              </div>
            </div>
          </section>

          <section className="mb-4">
            <h3 className="page-title">Quick Features</h3>
            <div className="row g-3">
              <Feature title="Responsive" text="Bootstrap ভিত্তিক responsive ডিজাইন।" />
              <Feature title="React Components" text="পেজগুলো আলাদা React components হিসেবে সাজানো রয়েছে।" />
              <Feature title="Simple Routing" text="ইন-ব্রাউজার নেভিগেশন: ক্লিক করে পেজ বদলাবে (single-file demo)।" />
            </div>
          </section>

          <section>
            <h3 className="page-title">Latest Posts</h3>
            <div className="row">
              {[1,2,3].map(i=> (
                <div className="col-md-4" key={i}>
                  <div className="card card-hover">
                    <img src={`https://picsum.photos/seed/post${i}/600/300`} className="card-img-top" alt="post" />
                    <div className="card-body">
                      <h5 className="card-title">Sample Post {i}</h5>
                      <p className="card-text">একটি উদাহরণ পোস্ট; আপনি এখানে ব্লগের পোস্ট বা নিউজ দেখাতে পারবেন।</p>
                      <button className="btn btn-sm btn-primary" onClick={()=>onNavigate('Blog')}>Read more</button>
                    </div>
                  </div>
                </div>
              ))}
            </div>
          </section>
        </div>
      )
    }

    function Feature({title,text}){
      return (
        <div className="col-md-4">
          <div className="p-3 bg-white rounded shadow-sm h-100">
            <h5>{title}</h5>
            <p className="mb-0">{text}</p>
          </div>
        </div>
      )
    }

    function About(){ const [content] = usePageContent('About'); return (
      <div className="card p-4">
        <h2>{content.title}</h2>
        <p>{content.body}</p>
      </div>
    ) }

    function Services(){ const [content] = usePageContent('Services'); return (
      <div>
        <h2>{content.title}</h2>
        <div className="row g-3">
          <div className="col-md-4"><div className="card p-3 h-100"><h5>Consulting</h5><p>কাস্টম কনসলটিং সার্ভিস</p></div></div>
          <div className="col-md-4"><div className="card p-3 h-100"><h5>Development</h5><p>ওয়েব ও মোবাইল ডেভেলপমেন্ট</p></div></div>
          <div className="col-md-4"><div className="card p-3 h-100"><h5>Training</h5><p>অনলাইন ও অফলাইন ট্রেনিং</p></div></div>
        </div>
      </div>
    ) }

    function Blog(){ const [content] = usePageContent('Blog'); return (
      <div>
        <h2>{content.title}</h2>
        <div className="list-group">
          {[1,2,3,4,5].map(i=> (
            <a key={i} href="#" className="list-group-item list-group-item-action" onClick={(e)=>e.preventDefault()}>
              <div className="d-flex w-100 justify-content-between">
                <h5 className="mb-1">Blog post #{i}</h5>
                <small>Oct {10+i}, 2025</small>
              </div>
              <p className="mb-1">এই একটি নমুনা ব্লগ পোস্টের সারাংশ।</p>
            </a>
          ))}
        </div>
      </div>
    ) }

    function Gallery(){ const [content] = usePageContent('Gallery'); return (
      <div>
        <h2>{content.title}</h2>
        <div className="row g-3">
          {Array.from({length:8}).map((_,i)=> (
            <div className="col-6 col-md-3" key={i}>
              <img src={`https://picsum.photos/seed/gallery${i}/400/300`} className="img-fluid rounded" alt={`g${i}`} />
            </div>
          ))}
        </div>
      </div>
    ) }

    function Team(){ const [content] = usePageContent('Team'); const members = ['Alice','Rahim','Mina','Sadia']; return (
      <div>
        <h2>{content.title}</h2>
        <div className="row g-3">
          {members.map((m,idx)=> (
            <div className="col-md-3" key={idx}>
              <div className="card text-center p-3">
                <img src={`https://i.pravatar.cc/150?img=${idx+3}`} className="rounded-circle mx-auto d-block" alt={m} />
                <h6 className="mt-2">{m}</h6>
                <small className="text-muted">Developer</small>
              </div>
            </div>
          ))}
        </div>
      </div>
    ) }

    function Resources(){ const [content] = usePageContent('Resources'); return (
      <div>
        <h2>{content.title}</h2>
        <ul>
          <li>Documentation</li>
          <li>Tutorials</li>
          <li>API Reference</li>
        </ul>
      </div>
    ) }

    function FAQ(){ const [content] = usePageContent('FAQ'); const faqs = [ {q:'কীভাবে সাইট ব্যবহার করব?', a:'নেভিগেশন ব্যবহার করে পেজগুলো দেখুন।'}, {q:'আমি কিভাবে যোগাযোগ করব?', a:'Contact পেইজে ফর্ম আছে।'} ]; return (
      <div>
        <h2>{content.title}</h2>
        {faqs.map((f,i)=> (
          <div key={i} className="mb-3"><h6>{f.q}</h6><p className="mb-0">{f.a}</p></div>
        ))}
      </div>
    ) }

    function Contact(){ const [content] = usePageContent('Contact'); const handleSubmit = (e)=>{ e.preventDefault(); alert('ধন্যবাদ! আপনার মেসেজ পাঠানো হয়েছে। (ডেমো)'); }
      return (
        <div className="card p-4">
          <h2>{content.title}</h2>
          <form onSubmit={handleSubmit}>
            <div className="mb-3">
              <label className="form-label">Name</label>
              <input className="form-control" required />
            </div>
            <div className="mb-3">
              <label className="form-label">Email</label>
              <input type="email" className="form-control" required />
            </div>
            <div className="mb-3">
              <label className="form-label">Message</label>
              <textarea className="form-control" rows="4" required></textarea>
            </div>
            <button className="btn btn-primary">Send Message</button>
          </form>
        </div>
      )
    }

    /*************** Authentication (Login / Signup) ***************/
    function Auth({setCurrentUser, onSuccess}){
      const [isLogin, setIsLogin] = useState(true);
      const [id, setId] = useState('');
      const [password, setPassword] = useState('');
      const [name, setName] = useState('');

      function handleLogin(e){
        e.preventDefault();
        const users = LS.get('wk_users', []);
        const u = users.find(x=> x.id === id && x.password === password);
        if(u){
          const userSafe = { id: u.id, name: u.name, role: u.role };
          setCurrentUser(userSafe);
          alert('লগইন সফল');
          if(onSuccess) onSuccess();
        }else{
          alert('ক্রেডেনশিয়াল ভুল অথবা ইউজার নেই');
        }
      }

      function handleSignup(e){
        e.preventDefault();
        const users = LS.get('wk_users', []);
        if(users.find(x=>x.id===id)){ alert('এই আইডি ইতিমধ্যে আছে'); return; }
        const newUser = { id, password, name: name||id, role: 'editor' };
        users.push(newUser);
        LS.set('wk_users', users);
        alert('রেজিস্ট্রেশন সফল — লগইন করুন');
        setIsLogin(true);
      }

      return (
        <div className="row justify-content-center">
          <div className="col-md-6">
            <div className="card p-4">
              <h3>{isLogin? 'Login' : 'Signup'}</h3>
              <form onSubmit={isLogin? handleLogin : handleSignup}>
                <div className="mb-3">
                  <label className="form-label">User ID</label>
                  <input className="form-control" value={id} onChange={e=>setId(e.target.value)} required />
                </div>
                {!isLogin && (
                  <div className="mb-3"><label className="form-label">Name</label><input className="form-control" value={name} onChange={e=>setName(e.target.value)} /></div>
                )}
                <div className="mb-3">
                  <label className="form-label">Password</label>
                  <input type="password" className="form-control" value={password} onChange={e=>setPassword(e.target.value)} required />
                </div>
                <div className="d-flex justify-content-between align-items-center">
                  <button className="btn btn-primary" type="submit">{isLogin? 'Login' : 'Signup'}</button>
                  <button className="btn btn-link" type="button" onClick={()=>setIsLogin(!isLogin)}>{isLogin? 'Create an account' : 'Have an account? Login'}</button>
                </div>
              </form>

              <hr />
              <small>ডিফল্ট অ্যাডমিন: id <code>admin</code> / pass <code>admin123</code></small>
            </div>
          </div>
        </div>
      )
    }

    /*************** Dashboard / Admin Editor ***************/
    function Dashboard({currentUser}){
      const pages = LS.get('wk_pages', defaultPages);
      const visitors = LS.get('wk_visitors', []);
      const users = LS.get('wk_users', []);
      const [selected, setSelected] = useState('Home');
      const [editTitle, setEditTitle] = useState(pages[selected].title);
      const [editBody, setEditBody] = useState(pages[selected].body);
      const [refresh, setRefresh] = useState(0);

      useEffect(()=>{
        const p = LS.get('wk_pages', defaultPages);
        setEditTitle(p[selected].title);
        setEditBody(p[selected].body);
      }, [selected, refresh]);

      function savePage(){
        const all = LS.get('wk_pages', defaultPages);
        all[selected] = { title: editTitle, body: editBody };
        LS.set('wk_pages', all);
        alert('Page saved');
        setRefresh(r=>r+1);
      }

      function exportVisitors(){
        const data = LS.get('wk_visitors', []);
        const csv = ['time,ua,language', ...data.map(d=> `${d.time.replace(/,/g,' ')},"${d.ua.replace(/"/g,'\"')}",${d.language||''}`)].join('\n');
        const blob = new Blob([csv], { type: 'text/csv' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a'); a.href = url; a.download = 'visitors.csv'; a.click(); URL.revokeObjectURL(url);
      }

      function clearVisitors(){ if(confirm('Visitor log মুছে ফেলবেন?')){ LS.set('wk_visitors', []); setRefresh(r=>r+1); } }

      return (
        <div>
          <h2>Dashboard</h2>
          <div className="row">
            <div className="col-md-4">
              <div className="p-3 bg-white rounded shadow-sm mb-3">Users: <strong>{users.length}</strong></div>
            </div>
            <div className="col-md-4">
              <div className="p-3 bg-white rounded shadow-sm mb-3">Visitors (total): <strong>{visitors.length}</strong></div>
            </div>
            <div className="col-md-4">
              <div className="p-3 bg-white rounded shadow-sm mb-3">Logged in as: <strong>{currentUser? currentUser.name : 'Guest'}</strong></div>
            </div>
          </div>

          <div className="row mt-3">
            <div className="col-md-4">
              <div className="card p-3">
                <h5>Pages</h5>
                <ul className="list-group">
                  {Object.keys(pages).map(p=> (
                    <li key={p} className={"list-group-item "+(p===selected? 'active' : '')} style={{cursor:'pointer'}} onClick={()=>setSelected(p)}>{p}</li>
                  ))}
                </ul>
              </div>

              <div className="card p-3 mt-3">
                <h6>Visitors</h6>
                <div className="d-flex gap-2">
                  <button className="btn btn-sm btn-outline-primary" onClick={exportVisitors}>Export CSV</button>
                  <button className="btn btn-sm btn-outline-danger" onClick={clearVisitors}>Clear</button>
                </div>
              </div>
            </div>

            <div className="col-md-8">
              <div className="card p-3">
                <h5>Editing: {selected}</h5>
                <div className="mb-3">
                  <label className="form-label">Title</label>
                  <input className="form-control" value={editTitle} onChange={e=>setEditTitle(e.target.value)} />
                </div>
                <div className="mb-3">
                  <label className="form-label">Body (HTML allowed)</label>
                  <textarea className="form-control editor" value={editBody} onChange={e=>setEditBody(e.target.value)} />
                </div>
                <div className="d-flex justify-content-between">
                  <div>
                    <button className="btn btn-primary me-2" onClick={savePage}>Save Page</button>
                    <button className="btn btn-secondary" onClick={()=>{ setEditTitle(pages[selected].title); setEditBody(pages[selected].body); }}>Revert</button>
                  </div>
                  <div>
                    <small className="text-muted">Tip: Site content saved in localStorage (wk_pages).</small>
                  </div>
                </div>

              </div>

              <div className="card p-3 mt-3">
                <h6>Recent Visitors (latest 10)</h6>
                <div style={{maxHeight:200, overflow:'auto'}}>
                  <ul className="list-group">
                    {visitors.slice(-10).reverse().map((v,i)=> (
                      <li key={i} className="list-group-item"><small>{v.time}</small><div><code style={{whiteSpace:'pre-wrap'}}>{v.ua}</code></div></li>
                    ))}
                    {visitors.length===0 && <li className="list-group-item">No visitors yet.</li>}
                  </ul>
                </div>
              </div>

            </div>
          </div>
        </div>
      )
    }

    /*************** Footer ***************/
    function Footer(){
      return (
        <footer className="mt-5">
          <div className="container d-flex justify-content-between align-items-center">
            <div>© {new Date().getFullYear()} <span className="logo">worldknows</span></div>
            <div>
              <a className="text-white me-3" href="#">Privacy</a>
              <a className="text-white" href="#">Terms</a>
            </div>
          </div>
        </footer>
      )
    }

    ReactDOM.createRoot(document.getElementById('root')).render(<App />)
  </script>

  <!-- Bootstrap JS (requires Popper) -->
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>

