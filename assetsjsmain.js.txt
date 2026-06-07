// --- Configuration & Constants ---
const CONFIG = {
    scrollThreshold: 60,
    typeSpeed: 80,
    deleteSpeed: 40,
    pauseBetween: 2000,
    roles: [
        "Mechanical Engineer",
        "Energy Systems Specialist",
        "HVAC Design Engineer",
        "CFD & FEA Analyst",
        "Machine Learning in Engineering",
        "Aerospace & UAV Engineer",
        "Sustainable Energy Engineer"
    ]
};

// --- Selectors ---
const selectors = {
    navbar: document.querySelector('.navbar'),
    navLinks: document.querySelector('.nav-links'),
    navItems: document.querySelectorAll('.nav-item'),
    mobileToggle: document.querySelector('.mobile-toggle'),
    themeToggle: document.querySelector('.theme-toggle'),
    fadeElements: document.querySelectorAll('.fade-up'),
    typedTarget: document.getElementById('typed-role'),
    sections: document.querySelectorAll('section[id]'),
    lazyImages: document.querySelectorAll('img[data-src]')
};

/**
 * Theme Management (Light/Dark Mode)
 */
const initTheme = () => {
    const savedTheme = localStorage.getItem('theme') || 'dark';
    document.documentElement.setAttribute('data-theme', savedTheme);
    
    if (selectors.themeToggle) {
        selectors.themeToggle.addEventListener('click', () => {
            const currentTheme = document.documentElement.getAttribute('data-theme');
            const newTheme = currentTheme === 'dark' ? 'light' : 'dark';
            
            document.documentElement.setAttribute('data-theme', newTheme);
            localStorage.setItem('theme', newTheme);
        });
    }
};

/**
 * Navbar Scroll Behavior
 */
const handleNavbarScroll = () => {
    if (!selectors.navbar) return;
    
    const scrollPosition = window.scrollY;
    if (scrollPosition > CONFIG.scrollThreshold) {
        selectors.navbar.classList.add('scrolled');
    } else {
        selectors.navbar.classList.remove('scrolled');
    }
};

/**
 * Mobile Menu Management
 */
const initMobileMenu = () => {
    if (!selectors.mobileToggle || !selectors.navLinks) return;

    selectors.mobileToggle.addEventListener('click', () => {
        const isExpanded = selectors.mobileToggle.getAttribute('aria-expanded') === 'true';
        selectors.mobileToggle.setAttribute('aria-expanded', !isExpanded);
        selectors.navLinks.classList.toggle('active');
        document.body.classList.toggle('menu-open');
    });

    // Close menu on link click
    selectors.navLinks.addEventListener('click', (e) => {
        if (e.target.classList.contains('nav-item')) {
            selectors.navLinks.classList.remove('active');
            selectors.mobileToggle.setAttribute('aria-expanded', 'false');
            document.body.classList.remove('menu-open');
        }
    });
};

/**
 * Intersection Observer: Fade-Up Animations
 */
const initFadeAnimations = () => {
    const observerOptions = {
        threshold: 0.15,
        rootMargin: '0px 0px -50px 0px'
    };

    const fadeObserver = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                entry.target.classList.add('visible');
                fadeObserver.unobserve(entry.target);
            }
        });
    }, observerOptions);

    selectors.fadeElements.forEach(el => fadeObserver.observe(el));
};

/**
 * Active Nav Link Highlighting
 */
const initActiveNav = () => {
    const navObserverOptions = {
        threshold: 0.5,
        rootMargin: '0px'
    };

    const navObserver = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                const id = entry.target.getAttribute('id');
                selectors.navItems.forEach(link => {
                    link.classList.remove('active');
                    if (link.getAttribute('href') === `#${id}`) {
                        link.classList.add('active');
                    }
                });
            }
        });
    }, navObserverOptions);

    selectors.sections.forEach(section => navObserver.observe(section));
};

/**
 * Custom Typed Text Effect (Vanilla JS)
 */
class TypedEffect {
    constructor(target, strings, options) {
        this.target = target;
        this.strings = strings;
        this.typeSpeed = options.typeSpeed;
        this.deleteSpeed = options.deleteSpeed;
        this.pause = options.pause;
        this.stringIdx = 0;
        this.charIdx = 0;
        this.isDeleting = false;
        this.init();
    }

    init() {
        if (!this.target) return;
        this.tick();
    }

    tick() {
        const currentString = this.strings[this.stringIdx];
        const fullText = this.isDeleting 
            ? currentString.substring(0, this.charIdx--) 
            : currentString.substring(0, this.charIdx++);

        this.target.textContent = fullText;

        let delta = this.isDeleting ? this.deleteSpeed : this.typeSpeed;

        if (!this.isDeleting && fullText === currentString) {
            delta = this.pause;
            this.isDeleting = true;
        } else if (this.isDeleting && fullText === '') {
            this.isDeleting = false;
            this.stringIdx = (this.stringIdx + 1) % this.strings.length;
            delta = 500;
        }

        setTimeout(() => this.tick(), delta);
    }
}

/**
 * Image Lazy Loading (Native + Polyfill support)
 */
const initLazyLoading = () => {
    if ('loading' in HTMLImageElement.prototype) {
        selectors.lazyImages.forEach(img => {
            img.src = img.dataset.src;
        });
    } else {
        const imgObserver = new IntersectionObserver((entries) => {
            entries.forEach(entry => {
                if (entry.isIntersecting) {
                    const img = entry.target;
                    img.src = img.dataset.src;
                    imgObserver.unobserve(img);
                }
            });
        });
        selectors.lazyImages.forEach(img => imgObserver.observe(img));
    }
};

/**
 * Smooth Scroll Polyfill/Handling
 */
const initSmoothScroll = () => {
    document.querySelectorAll('a[href^="#"]').forEach(anchor => {
        anchor.addEventListener('click', function (e) {
            const targetId = this.getAttribute('href');
            if (targetId === '#') return;
            
            const targetElement = document.querySelector(targetId);
            if (targetElement) {
                e.preventDefault();
                window.scrollTo({
                    top: targetElement.offsetTop - 80,
                    behavior: 'smooth'
                });
            }
        });
    });
};

// --- Execution ---
initTheme();
initMobileMenu();
initFadeAnimations();
initActiveNav();
initLazyLoading();
initSmoothScroll();

window.addEventListener('scroll', handleNavbarScroll, { passive: true });

if (selectors.typedTarget) {
    new TypedEffect(selectors.typedTarget, CONFIG.roles, {
        typeSpeed: CONFIG.typeSpeed,
        deleteSpeed: CONFIG.deleteSpeed,
        pause: CONFIG.pauseBetween
    });
}