/* =============================================================
   VETOCOS — Shared Supabase client + helpers
   Loaded by every page. Exposes window.Vetocos namespace.
   ============================================================= */

(function () {
  'use strict';

  // -----------------------------------------------------------
  // 1. CONFIG — your Supabase project
  // -----------------------------------------------------------
  const SUPABASE_URL = 'https://qdepgviimbcdlozjpbhm.supabase.co';
  const SUPABASE_ANON_KEY =
    'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InFkZXBndmlpbWJjZGxvempwYmhtIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTk0OTgwOTAsImV4cCI6MjA3NTA3NDA5MH0.-F95mmQw-pnsD-K8FIUtbj3oWeiPCSE0Pmuco9lYo_0';

  // Make sure the supabase-js library has loaded first
  if (typeof window.supabase === 'undefined') {
    console.error('[Vetocos] supabase-js library not loaded. Add this BEFORE this script: <script src="https://unpkg.com/@supabase/supabase-js@2"></script>');
    return;
  }

  const supa = window.supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY, {
    auth: {
      persistSession: true,
      autoRefreshToken: true,
      detectSessionInUrl: true,
    },
  });

  // -----------------------------------------------------------
  // 2. CACHE — current user + profile
  // -----------------------------------------------------------
  let cachedProfile = null;
  let cachedProfileFetchedAt = 0;
  const PROFILE_CACHE_MS = 30000; // 30 seconds

  // -----------------------------------------------------------
  // 3. AUTH METHODS
  // -----------------------------------------------------------
  async function signUp(email, password, metadata) {
    // metadata: { first_name, last_name, phone, country, clinic_name, role }
    const { data, error } = await supa.auth.signUp({
      email,
      password,
      options: {
        data: metadata || {},
        emailRedirectTo: window.location.origin + '/login.html',
      },
    });
    if (error) throw error;
    return data;
  }

  async function signIn(email, password) {
    const { data, error } = await supa.auth.signInWithPassword({ email, password });
    if (error) throw error;
    cachedProfile = null;
    return data;
  }

  async function signOut() {
    cachedProfile = null;
    const { error } = await supa.auth.signOut();
    if (error) throw error;
  }

  async function sendPasswordReset(email) {
    const { error } = await supa.auth.resetPasswordForEmail(email, {
      redirectTo: window.location.origin + '/update-password.html',
    });
    if (error) throw error;
  }

  async function updatePassword(newPassword) {
    const { error } = await supa.auth.updateUser({ password: newPassword });
    if (error) throw error;
  }

  // -----------------------------------------------------------
  // 4. SESSION + PROFILE
  // -----------------------------------------------------------
  async function getSession() {
    const { data } = await supa.auth.getSession();
    return data.session;
  }

  async function getUser() {
    const session = await getSession();
    return session ? session.user : null;
  }

  /**
   * Returns the current user's profile (with status, is_admin, etc).
   * Cached briefly to avoid hitting the DB on every page interaction.
   * Returns null if the user is logged out.
   */
  async function getProfile(force) {
    const session = await getSession();
    if (!session) {
      cachedProfile = null;
      return null;
    }
    const fresh = force === true || (Date.now() - cachedProfileFetchedAt) > PROFILE_CACHE_MS;
    if (cachedProfile && !fresh) return cachedProfile;

    const { data, error } = await supa
      .from('profiles')
      .select('id, email, first_name, last_name, phone, country, clinic_name, role, status, is_admin, rejection_reason, created_at, approved_at')
      .eq('id', session.user.id)
      .maybeSingle();

    if (error) {
      console.warn('[Vetocos] Failed to load profile', error);
      return null;
    }
    cachedProfile = data;
    cachedProfileFetchedAt = Date.now();
    return data;
  }

  /**
   * Subscribe to login/logout events. Callback receives ({ user, profile }).
   * Useful for the header to update when the user signs in/out.
   */
  function onAuthChange(callback) {
    return supa.auth.onAuthStateChange(async (_event, session) => {
      cachedProfile = null;
      const profile = session ? await getProfile(true) : null;
      try { callback({ user: session ? session.user : null, profile: profile }); }
      catch (e) { console.error('[Vetocos] auth listener error', e); }
    });
  }

  // -----------------------------------------------------------
  // 5. PAGE GUARDS — call on page load to enforce access
  // -----------------------------------------------------------

  /** Redirects to login if user is not signed in. Returns the user when ok. */
  async function requireAuth(returnUrl) {
    const user = await getUser();
    if (!user) {
      const from = returnUrl || (location.pathname + location.search + location.hash);
      location.href = 'login.html?redirect=' + encodeURIComponent(from);
      return null;
    }
    return user;
  }

  /** Redirects to login or home if user is not an admin. Returns profile when ok. */
  async function requireAdmin() {
    await requireAuth();
    const profile = await getProfile(true);
    if (!profile || !profile.is_admin) {
      toast('You don\'t have permission to view this page.', 'error');
      setTimeout(function () { location.href = 'index.html'; }, 1200);
      return null;
    }
    return profile;
  }

  // -----------------------------------------------------------
  // 6. TOAST NOTIFICATIONS
  // -----------------------------------------------------------
  function ensureToastContainer() {
    let c = document.querySelector('.toast-container');
    if (!c) {
      c = document.createElement('div');
      c.className = 'toast-container';
      document.body.appendChild(c);
    }
    return c;
  }

  /**
   * Show a toast. type: 'success' | 'error' | 'warning' | 'info' (default).
   */
  function toast(message, type) {
    const container = ensureToastContainer();
    const el = document.createElement('div');
    el.className = 'toast' + (type ? ' toast-' + type : '');
    el.textContent = message;
    container.appendChild(el);
    setTimeout(function () {
      el.style.transition = 'opacity 200ms ease, transform 200ms ease';
      el.style.opacity = '0';
      el.style.transform = 'translateX(20px)';
      setTimeout(function () { el.remove(); }, 220);
    }, 4500);
  }

  // -----------------------------------------------------------
  // 7. HELPERS
  // -----------------------------------------------------------

  /** Build a public URL for a file in the ProductFiles storage bucket. */
  function storageUrl(path) {
    if (!path) return '';
    const clean = String(path).replace(/^ProductFiles\//, '');
    return SUPABASE_URL + '/storage/v1/object/public/ProductFiles/' + encodeURI(clean);
  }

  /** Format a price for display. */
  function formatPrice(amount, currency) {
    if (amount == null || isNaN(amount)) return '—';
    const cur = currency || 'MAD';
    return Number(amount).toFixed(2) + ' ' + cur;
  }

  /** Convenience: read ?key= from URL. */
  function getQueryParam(key) {
    const params = new URLSearchParams(location.search);
    return params.get(key);
  }

  /** Escape user-provided strings before inserting into HTML. */
  function escapeHtml(str) {
    if (str == null) return '';
    return String(str)
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#39;');
  }

  // -----------------------------------------------------------
  // 8. EXPORT
  // -----------------------------------------------------------
  window.Vetocos = {
    supa: supa,
    signUp: signUp,
    signIn: signIn,
    signOut: signOut,
    sendPasswordReset: sendPasswordReset,
    updatePassword: updatePassword,
    getSession: getSession,
    getUser: getUser,
    getProfile: getProfile,
    onAuthChange: onAuthChange,
    requireAuth: requireAuth,
    requireAdmin: requireAdmin,
    toast: toast,
    storageUrl: storageUrl,
    formatPrice: formatPrice,
    getQueryParam: getQueryParam,
    escapeHtml: escapeHtml,
  };
})();
