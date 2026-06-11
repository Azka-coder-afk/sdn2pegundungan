# bagian firebase jgn lupa diganti

/* ════════════════════════════════════════
   0. FIREBASE — REAL-TIME 404 BROADCAST
   ──────────────────────────────────────
   CARA SETUP (wajib diisi sebelum deploy):
   1. Buka https://console.firebase.google.com
   2. Buat project baru → klik "Add app" → pilih Web (</>)
   3. Copy nilai dari firebaseConfig yang dikasih Firebase
   4. Tempel di bawah (ganti yang bertanda GANTI_INI)
   5. Di Firebase Console → Realtime Database → Create database
      → pilih mode "Start in test mode" (bisa baca-tulis semua)
════════════════════════════════════════ */
(function () {
  'use strict';

  /* ► GANTI BAGIAN INI dengan config Firebase kamu ◄ */
  const FIREBASE_CONFIG = {
    apiKey:            "GANTI_INI",
    authDomain:        "GANTI_INI.firebaseapp.com",
    databaseURL:       "https://GANTI_INI-default-rtdb.firebaseio.com",
    projectId:         "GANTI_INI",
    storageBucket:     "GANTI_INI.firebasestorage.app",
    messagingSenderId: "GANTI_INI",
    appId:             "GANTI_INI"
  };

  /* Cek apakah config sudah diisi */
  const isConfigured = !Object.values(FIREBASE_CONFIG).some(v => v.includes('GANTI_INI'));

  if (!isConfigured) {
    console.warn('[SDN2PGD] Firebase belum dikonfigurasi. Fitur 404 broadcast dinonaktifkan.');
    window._firebaseReady = false;
  } else {
    try {
      const app = firebase.initializeApp(FIREBASE_CONFIG);
      const db  = firebase.database(app);
      const ref = db.ref('sdn2pgd/show404');

      /* ── LISTEN: semua device memantau node ini ── */
      ref.on('value', (snap) => {
        const val = snap.val();
        const overlay = document.getElementById('error-404-overlay');
        if (!overlay) return;

        if (val === true) {
          overlay.classList.add('show');
          // Update URL tanpa reload
          if (location.hash !== '#404') history.pushState({ page404: true }, '', '#404');
        } else {
          overlay.classList.remove('show');
          if (location.hash === '#404') history.replaceState(null, '', location.pathname);
        }
      });

      /* ── EXPOSE: fungsi untuk set Firebase dari mana saja ── */
      window._firebase404Show = () => ref.set(true);
      window._firebase404Hide = () => ref.set(false);
      window._firebaseReady   = true;

      console.info('[SDN2PGD] Firebase terhubung. 404 broadcast aktif.');
    } catch (err) {
      console.error('[SDN2PGD] Firebase error:', err.message);
      window._firebaseReady = false;
    }
  }

})();
