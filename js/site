/* =============================================================
   VETOCOS — Site chrome (header + status banner + footer)
   Pages drop a <div id="header-mount"></div> and a
   <div id="footer-mount"></div>, then call:
     Vetocos.mountHeader('#header-mount');
     Vetocos.mountFooter('#footer-mount');
   ============================================================= */

(function () {
  'use strict';

  if (!window.Vetocos) {
    console.error('[Vetocos] vetocos.js must load BEFORE site.js');
    return;
  }

  const V = window.Vetocos;
  const SUPER_ADMIN_EMAILS = ['Anwar.moumenjamai19@gmail.com'];

  // -----------------------------------------------------------
  // HEADER MARKUP
  // -----------------------------------------------------------
  function headerHtml() {
    return `
      <header class="site-header">
        <div class="container">
          <a class="site-brand" href="index.html" aria-label="VetoCos home">
            <img src="images/logo.png" alt="" onerror="this.style.display='none'">
            <span>VetoCos</span>
          </a>

          <nav class="site-nav" id="siteNav" aria-label="Primary navigation">
            <a href="index.html" data-nav="home">Home</a>
            <a href="index.html#products" data-nav="products">Products</a>
            <a href="news.html" data-nav="news">News</a>
            <a href="vet-verify.html" data-nav="vets">For Vets</a>

            <span class="nav-auth-area" id="navAuthArea">
              <!-- filled in by updateHeaderState() -->
            </span>
          </nav>

          <button class="nav-toggle" id="navToggle" aria-label="Open menu" aria-expanded="false">☰</button>
        </div>
      </header>

      <div id="statusBannerMount"></div>
    `;
  }

  // -----------------------------------------------------------
  // RIGHT-SIDE AUTH BUTTONS / USER CHIP
  // -----------------------------------------------------------
  function loggedOutHtml() {
    return `
      <a class="btn btn-ghost btn-sm" href="login.html">Sign in</a>
      <a class="btn btn-sm" href="signup.html">Sign up</a>
    `;
  }

  function loggedInHtml(profile) {
    const name = (profile.first_name || profile.email || 'Account').trim();
    const initials = (name[0] || '?').toUpperCase();
    const showAdmin = profile.is_admin === true;

    return `
      <button class="user-chip" id="userChip" aria-haspopup="menu" aria-expanded="false">
        <span class="user-avatar">${V.escapeHtml(initials)}</span>
        <span>${V.escapeHtml(name)}</span>
      </button>
      <div class="user-menu" id="userMenu" role="menu">
        <div class="user-menu-header">
          <strong>${V.escapeHtml(name)}</strong>
          <div class="text-soft text-xsmall">${V.escapeHtml(profile.email || '')}</div>
          ${profileStatusBadge(profile)}
        </div>
        <a href="account.html" role="menuitem">Account settings</a>
        <a href="my-orders.html" role="menuitem">My orders</a>
        ${showAdmin ? `<a href="admin.html" role="menuitem"><strong>Admin panel</strong></a>` : ''}
        <button type="button" id="signOutBtn" role="menuitem">Sign out</button>
      </div>
    `;
  }

  function profileStatusBadge(profile) {
    if (profile.is_admin) return `<span class="badge badge-primary mt-2">Admin</span>`;
    if (profile.status === 'approved') return `<span class="badge badge-success mt-2">Approved</span>`;
    if (profile.status === 'rejected') return `<span class="badge badge-danger mt-2">Rejected</span>`;
    return `<span class="badge badge-warning mt-2">Pending approval</span>`;
  }

  // -----------------------------------------------------------
  // STATUS BANNER (pending / rejected)
  // -----------------------------------------------------------
  function statusBannerHtml(profile) {
    if (!profile) return '';
    if (profile.is_admin) return '';
    if (profile.status === 'pending') {
      return `
        <div class="status-banner">
          <strong>Your account is pending approval.</strong>
          You can browse our products, but prices and ordering are unlocked once our team approves your account.
        </div>
      `;
    }
    if (profile.status === 'rejected') {
      const reason = profile.rejection_reason ? ' Reason: ' + V.escapeHtml(profile.rejection_reason) : '';
      return `
        <div class="status-banner rejected">
          <strong>Your account was not approved.</strong>${reason}
          Please <a href="#" onclick="document.getElementById('contactModal')?.classList.add('open');return false;">contact us</a> if you think this is a mistake.
        </div>
      `;
    }
    return '';
  }

  // -----------------------------------------------------------
  // STATE UPDATE — call whenever auth changes
  // -----------------------------------------------------------
  async function updateHeaderState() {
    const authArea = document.getElementById('navAuthArea');
    const bannerMount = document.getElementById('statusBannerMount');
    if (!authArea) return;

    const profile = await V.getProfile(true);

    if (!profile) {
      authArea.innerHTML = loggedOutHtml();
      if (bannerMount) bannerMount.innerHTML = '';
      return;
    }

    authArea.innerHTML = loggedInHtml(profile);
    if (bannerMount) bannerMount.innerHTML = statusBannerHtml(profile);

    wireUserMenu();
  }

  // -----------------------------------------------------------
  // EVENT WIRING
  // -----------------------------------------------------------
  function wireMobileMenu() {
    const toggle = document.getElementById('navToggle');
    const nav = document.getElementById('siteNav');
    if (!toggle || !nav) return;
    toggle.addEventListener('click', function () {
      const open = nav.classList.toggle('open');
      toggle.setAttribute('aria-expanded', open ? 'true' : 'false');
    });
    // Close mobile nav when clicking a link inside it
    nav.addEventListener('click', function (e) {
      if (e.target.closest('a')) nav.classList.remove('open');
    });
  }

  function wireUserMenu() {
    const chip = document.getElementById('userChip');
    const menu = document.getElementById('userMenu');
    const signOutBtn = document.getElementById('signOutBtn');
    if (!chip || !menu) return;

    chip.addEventListener('click', function (e) {
      e.stopPropagation();
      const open = menu.classList.toggle('open');
      chip.setAttribute('aria-expanded', open ? 'true' : 'false');
    });

    document.addEventListener('click', function (e) {
      if (!menu.contains(e.target) && !chip.contains(e.target)) {
        menu.classList.remove('open');
        chip.setAttribute('aria-expanded', 'false');
      }
    });

    if (signOutBtn) {
      signOutBtn.addEventListener('click', async function () {
        try {
          await V.signOut();
          V.toast('Signed out', 'success');
          setTimeout(function () { location.href = 'index.html'; }, 600);
        } catch (e) {
          V.toast(e.message || 'Could not sign out', 'error');
        }
      });
    }
  }

  function highlightActiveNav() {
    const path = (location.pathname.split('/').pop() || 'index.html').toLowerCase();
    const map = { '': 'home', 'index.html': 'home', 'news.html': 'news', 'vet-verify.html': 'vets' };
    const key = map[path];
    if (!key) return;
    const link = document.querySelector('.site-nav a[data-nav="' + key + '"]');
    if (link) link.classList.add('active');
  }

  // -----------------------------------------------------------
  // FOOTER
  // -----------------------------------------------------------
  function footerHtml() {
    const year = new Date().getFullYear();
    return `
      <footer class="site-footer">
        <div class="container">
          <div class="footer-grid">
            <div>
              <h4>VetoCos</h4>
              <p class="text-small" style="color:rgba(255,255,255,.65); max-width:300px;">
                Science-guided complementary feed for dogs &amp; cats. Developed with veterinarians.
              </p>
              <p class="text-xsmall" style="color:rgba(255,255,255,.55); margin-top:12px;">
                Complementary feed — not a medicine. Always follow your veterinarian's advice.
              </p>
            </div>

            <div>
              <h4>Explore</h4>
              <ul class="footer-links">
                <li><a href="index.html">Home</a></li>
                <li><a href="index.html#products">Products</a></li>
                <li><a href="news.html">News</a></li>
                <li><a href="vet-verify.html">For Vets</a></li>
              </ul>
            </div>

            <div>
              <h4>Account</h4>
              <ul class="footer-links">
                <li><a href="login.html">Sign in</a></li>
                <li><a href="signup.html">Create account</a></li>
                <li><a href="forgot-password.html">Forgot password</a></li>
              </ul>
            </div>

            <div>
              <h4>Contact</h4>
              <ul class="footer-links">
                <li><a href="mailto:info@vetocos.com">info@vetocos.com</a></li>
                <li><a href="tel:+212537742213">+212 537 742 213</a></li>
                <li><a href="https://wa.me/212537742213" target="_blank" rel="noopener">WhatsApp</a></li>
              </ul>
            </div>
          </div>

          <div class="footer-bottom">
            <span>© ${year} VetoCos. All rights reserved.</span>
            <span>Complementary feed for companion animals</span>
          </div>
        </div>
      </footer>
    `;
  }

  // -----------------------------------------------------------
  // PUBLIC API
  // -----------------------------------------------------------
  V.mountHeader = function (selector) {
    const target = typeof selector === 'string' ? document.querySelector(selector) : selector;
    if (!target) {
      console.warn('[Vetocos] mountHeader: target not found', selector);
      return;
    }
    target.outerHTML = headerHtml();
    wireMobileMenu();
    highlightActiveNav();
    updateHeaderState();          // initial render

    // Listen for auth state changes (login, logout, etc.)
    V.onAuthChange(function () { updateHeaderState(); });
  };

  V.mountFooter = function (selector) {
    const target = typeof selector === 'string' ? document.querySelector(selector) : selector;
    if (!target) {
      console.warn('[Vetocos] mountFooter: target not found', selector);
      return;
    }
    target.outerHTML = footerHtml();
  };

  // Convenience to refresh the header without a full page reload (e.g. after admin approves themselves)
  V.refreshHeader = updateHeaderState;
})();
