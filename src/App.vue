<script setup>
import { ref, reactive, computed, onMounted, onUnmounted } from "vue";

/* ---------------------------------------------------------------
   Cursor follower (dot + ring + ambient glow)
--------------------------------------------------------------- */
const cursorDot = ref(null);
const cursorRing = ref(null);
const cursorGlow = ref(null);
const ringActive = ref(false);

let raf = null;
const mouse = { x: 0, y: 0 };
const ringPos = { x: 0, y: 0 };

function onMouseMove(e) {
  mouse.x = e.clientX;
  mouse.y = e.clientY;
  if (cursorDot.value) cursorDot.value.style.transform = `translate(${mouse.x}px, ${mouse.y}px) translate(-50%, -50%)`;
  if (cursorGlow.value) cursorGlow.value.style.transform = `translate(${mouse.x}px, ${mouse.y}px) translate(-50%, -50%)`;
}
function animateRing() {
  ringPos.x += (mouse.x - ringPos.x) * 0.18;
  ringPos.y += (mouse.y - ringPos.y) * 0.18;
  if (cursorRing.value) cursorRing.value.style.transform = `translate(${ringPos.x}px, ${ringPos.y}px) translate(-50%, -50%)`;
  raf = requestAnimationFrame(animateRing);
}
function activateRing() { ringActive.value = true; }
function deactivateRing() { ringActive.value = false; }

/* ---------------------------------------------------------------
   3D tilt on glass cards
--------------------------------------------------------------- */
function tiltMove(e) {
  const card = e.currentTarget;
  const rect = card.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;
  const rotateX = ((y - rect.height / 2) / (rect.height / 2)) * -6;
  const rotateY = ((x - rect.width / 2) / (rect.width / 2)) * 6;
  card.style.transform = `perspective(900px) rotateX(${rotateX}deg) rotateY(${rotateY}deg) translateY(-4px)`;
  card.style.setProperty("--mouse-x", `${x}px`);
  card.style.setProperty("--mouse-y", `${y}px`);
}
function tiltLeave(e) {
  e.currentTarget.style.transform = "perspective(900px) rotateX(0deg) rotateY(0deg) translateY(0)";
}

/* ---------------------------------------------------------------
   Nav scroll hide/show
--------------------------------------------------------------- */
const navEl = ref(null);
let lastScroll = 0;
function onScroll() {
  const current = window.pageYOffset;
  if (!navEl.value) return;
  navEl.value.style.transform = current > lastScroll && current > 100 ? "translateY(-100%)" : "translateY(0)";
  lastScroll = current;
}
function scrollToId(id) {
  document.getElementById(id)?.scrollIntoView({ behavior: "smooth" });
}

/* ---------------------------------------------------------------
   Stat counters
--------------------------------------------------------------- */
const stats = reactive([
  { icon: "rocket_launch", target: 10, value: 0, label: "Projects Completed", color: "secondary" },
  { icon: "sentiment_very_satisfied", target: 3, value: 0, label: "Happy Clients", color: "tertiary" },
  { icon: "trending_up", target: 90, value: 0, label: "Avg. Growth Delivered %", color: "primary" },
]);
let countersStarted = false;
function animateCounters() {
  if (countersStarted) return;
  countersStarted = true;
  stats.forEach((stat) => {
    const step = () => {
      const inc = stat.target / 100;
      if (stat.value < stat.target) {
        stat.value = Math.min(stat.target, Math.ceil(stat.value + inc));
        setTimeout(step, 12);
      } else stat.value = stat.target;
    };
    step();
  });
}

/* ---------------------------------------------------------------
   Skill bars (bento panels)
--------------------------------------------------------------- */
const frontendSkills = [
  { name: "HTML5", pct: 98 },
  { name: "CSS3", pct: 95 },
  { name: "JavaScript", pct: 92 },
  { name: "Vue.js", pct: 90 },
  { name: "Bootstrap", pct: 98 },
  { name: "Tailwind CSS", pct: 90 },
  { name: "Responsive Design", pct: 98 },
  { name: "Vite", pct: 90 },
];
const backendSkills = [
  { name: "Laravel", pct: 95 },
  { name: "PHP", pct: 95 },
  { name: "MySQL", pct: 93 },
  { name: "REST API", pct: 92 },
  { name: "Authentication", pct: 90 },
  { name: "Laravel Sanctum", pct: 90 },
  { name: "CRUD Operations", pct: 98 },
  { name: "Git & GitHub", pct: 88 },
];
const barsRevealed = ref(false);
function revealBars() { barsRevealed.value = true; }

/* ---------------------------------------------------------------
   360° skill carousel — auto-rotating ring of skill cards.
   Pauses smoothly on hover, resumes on leave.
--------------------------------------------------------------- */
const skillRing = [
  { name: "Vue.js", icon: "view_quilt", color: "secondary" },
  { name: "Laravel", icon: "dns", color: "tertiary" },
  { name: "PHP", icon: "integration_instructions", color: "primary" },
  { name: "MySQL", icon: "database", color: "secondary" },
  { name: "Tailwind CSS", icon: "palette", color: "tertiary" },
  { name: "Bootstrap", icon: "grid_view", color: "primary" },
  { name: "Flutter", icon: "smartphone", color: "secondary" },
  { name: "Shopify", icon: "storefront", color: "tertiary" },
  { name: "WordPress", icon: "language", color: "primary" },
  { name: "Wix", icon: "web", color: "secondary" },
  { name: "REST APIs", icon: "hub", color: "tertiary" },
  { name: "Git", icon: "terminal", color: "primary" },
];
const CAROUSEL_RADIUS = 340;
const carouselItems = computed(() =>
  skillRing.map((s, i) => {
    const angle = (360 / skillRing.length) * i;
    return { ...s, style: `transform: rotateY(${angle}deg) translateZ(${CAROUSEL_RADIUS}px);` };
  })
);
const carouselAngle = ref(0);
const carouselPaused = ref(false);
let carouselRaf = null;
function spinCarousel() {
  if (!carouselPaused.value) carouselAngle.value += 0.045;
  carouselRaf = requestAnimationFrame(spinCarousel);
}

/* ---------------------------------------------------------------
   Content data
--------------------------------------------------------------- */
const services = [
  { icon: "layers", color: "secondary", title: "Custom Web Apps", desc: "Scalable, conversion-focused applications built with Laravel, Vue.js, and cloud-native architectures.", tags: ["Vue.js", "Laravel"] },
  { icon: "storefront", color: "tertiary", title: "E-Commerce & Stores", desc: "Revenue-ready storefronts on Shopify, WordPress/WooCommerce, and custom carts that convert.", tags: ["Shopify", "WordPress"] },
  { icon: "hub", color: "secondary-fixed", title: "API Integration", desc: "Seamless third-party connectivity and robust RESTful microservices that keep systems talking.", tags: ["REST", "Sanctum"] },
  { icon: "speed", color: "primary", title: "Growth & Optimization", desc: "Core Web Vitals, SEO foundations, and query tuning aimed squarely at business growth, not vanity metrics.", tags: ["SEO", "WebPerf"] },
];

const experience = [
 {
  year: "January 4, 2025",
  role: "Full Stack Developer",
  company: "ODShops",
  color: "secondary",

  points: [
    "Created a custom-designed eCommerce platform using Laravel and Vue.js.",
    "Integrated external APIs for product synchronization and order management.",
    "Developed responsive, high-performance frontend components with Vue.js.",
    "Optimized backend functionality and database performance for scalability.",
  ],
},
 {
  year: "2020 — 2022",
  role: "Full Stack Developer",
  company: "Client Projects",
  color: "tertiary",

  points: [
    "Developed FILTERMART (filtermart.online), a complete eCommerce platform using Laravel with responsive and mobile-friendly Blade-based UI design.",
    "Implemented secure authentication using Laravel Sanctum API with role-based access and protected user sessions.",
    "Built advanced product search, dynamic filters, categories, and sorting features to improve customer shopping experience.",
    "Designed and optimized database architecture using Laravel Eloquent model relationships for products, categories, orders, users, and inventory management.",
    "Developed a scalable admin panel to manage products, orders, customers, categories, and website content.",
    "Integrated modern UI components, optimized performance, and ensured cross-browser compatibility across desktop, tablet, and mobile devices.",
  ],
},
 {
  year: "Earlier",
  role: "Full Stack Developer",
  company: "EvGoo — eBike Rental & Sales Platform",
  color: "primary",

  points: [
    "Developed a complete full-stack eBike rental and sales platform using Laravel with a modern and scalable architecture.",
    "Built features for bike listings, rental management, sales workflow, customer accounts, orders, and admin management.",
    "Designed responsive user interfaces and optimized the platform for desktop, tablet, and mobile devices.",
    "Implemented Laravel backend logic, database relationships, authentication, and management systems for seamless operations.",
    "Created an admin panel for managing bikes, bookings, customers, payments, and platform data.",
    "Completed full development lifecycle; the project is finalized and prepared for deployment.",
  ],
},
{
  year: "Earlier",
  role: "Full Stack Developer",
  company: "Law Firm — Legal Management Platform",
  color: "primary",

  points: [
    "Developed a complete full-stack law firm website and management platform using Laravel with Blade templating.",
    "Built a modern, responsive frontend experience for clients with optimized UI/UX across desktop and mobile devices.",
    "Implemented Laravel APIs for dynamic data management and seamless communication between frontend and backend systems.",
    "Created a powerful admin panel dashboard to manage services, attorneys, cases, clients, appointments, and website content.",
    "Designed database architecture with Laravel Eloquent relationships and secure authentication for admin operations.",
    "Developed scalable backend functionality with clean code structure, validation, and optimized performance.",
  ],
},
{
  year: "Earlier",
  role: "Full Stack Developer",
  company: "AptCompressor.com — Full Stack Web Platform",
  color: "primary",

  points: [
    "Developed a complete full-stack web platform for AptCompressor.com using Laravel with a modern, scalable architecture.",
    "Designed and developed a fully responsive frontend interface optimized for desktop, tablet, and mobile devices.",
    "Built dynamic website modules including products/services management, content sections, contact systems, and business workflows.",
    "Created a powerful admin panel dashboard to manage website content, products, inquiries, users, and platform settings.",
    "Implemented Laravel backend functionality with secure authentication, API integration, database optimization, and Eloquent model relationships.",
    "Developed reusable components, clean code structure, and performance optimizations to ensure a fast and maintainable application.",
    "Handled complete project development lifecycle including UI design, backend development, database management, testing, and deployment preparation.",
  ],
},
{
  year: "Earlier",
  role: "Full Stack Developer",
  company: "DrMNawazAnjum.com — Medical Appointment Platform",
  color: "primary",

  points: [
    "Developed a complete full-stack healthcare website for Dr. M. Nawaz Anjum using Laravel with a modern and scalable architecture.",
    "Designed and developed a responsive frontend using Laravel Blade templating with a user-friendly experience across desktop and mobile devices.",
    "Built an online appointment booking system allowing patients to select available dates and time slots for consultations.",
    "Implemented appointment scheduling logic with time-slot management, booking validation, and user request handling.",
    "Created a powerful admin panel dashboard to manage appointments, patients, doctor information, services, schedules, and website content.",
    "Developed secure backend functionality with Laravel authentication, database relationships, form validation, and optimized queries.",
    "Integrated APIs for dynamic data handling and created a maintainable system for future scalability and feature expansion.",
    "Managed the complete development lifecycle including UI design, backend development, database structure, testing, and deployment preparation.",
  ],
},
];

const projects = [
  {
    tag: "SaaS Multi-Tool",
    title: "NexoTools",
    desc: "A QR code generator and file conversion suite (PDF, Word, Excel, image) built with a scalable SaaS architecture.",
    stack: ["Laravel 10", "Vue 3", "Vite"],
    color: "secondary",
    link: null,
    button: "GitHub",
  },

  {
    tag: "E-Learning Platform",
    title: "SabaSaeed",
    desc: "A complete student and teacher management platform with OTP authentication, role-based permissions, and a powerful admin dashboard.",
    stack: ["Laravel", "Sanctum", "Vue.js", "MySQL"],
    color: "tertiary",
    link: null,
    button: "GitHub",
  },

  {
    tag: "E-Commerce",
    title: "FilterMart",
    desc: "A complete eCommerce platform with responsive Laravel Blade design, secure authentication, Sanctum API integration, advanced search filters, categories, and Eloquent model relationships.",
    stack: ["Laravel", "Blade", "Sanctum", "MySQL"],
    color: "primary",
    link: "https://filtermart.online/",
    button: "Live Demo",
  },

  {
    tag: "eBike Rental & Sales",
    title: "EvGoo",
    desc: "A full-stack eBike rental and sales platform with bike listings, rental management, customer accounts, order workflow, and admin management system.",
    stack: ["Laravel", "Vue.js", "MySQL", "REST API"],
    color: "secondary",
    link: null,
    button: "GitHub",
  },

  {
    tag: "Legal Platform",
    title: "Law Firm Management Platform",
    desc: "A complete law firm website and management system with Laravel Blade frontend, API integration, authentication, and admin dashboard for managing services, attorneys, cases, and clients.",
    stack: ["Laravel", "Blade", "API", "MySQL"],
    color: "tertiary",
    link: null,
    button: "GitHub",
  },

  {
    tag: "Business Website",
    title: "AptCompressor",
    desc: "A full-stack business platform with custom UI design, Laravel backend, API integration, responsive frontend, and a complete admin panel for managing website content and business data.",
    stack: ["Laravel", "Blade", "API", "Admin Panel"],
    color: "primary",
    link: "https://aptcompressor.com/",
    button: "Live Demo",
  },

  {
    tag: "Healthcare Appointment",
    title: "Dr. M. Nawaz Anjum",
    desc: "A healthcare appointment booking platform with online scheduling, time-slot selection, patient management, doctor profile management, and a complete admin dashboard.",
    stack: ["Laravel", "Blade", "MySQL", "Admin Panel"],
    color: "secondary",
    link: "https://drmnawazanjum.com/",
    button: "Live Demo",
  },

  {
    tag: "Custom Development",
    title: "Client Projects",
    desc: "Delivered custom Laravel, Vue.js, WordPress, and Shopify solutions for businesses with responsive designs and scalable backend systems.",
    stack: ["Laravel", "Vue.js", "WordPress", "Shopify"],
    color: "tertiary",
    link: null,
    button: "GitHub",
  },
];

onMounted(() => {
  window.addEventListener("mousemove", onMouseMove, { passive: true });
  window.addEventListener("scroll", onScroll, { passive: true });
  raf = requestAnimationFrame(animateRing);
  carouselRaf = requestAnimationFrame(spinCarousel);

  document.querySelectorAll("a, button, .tilt-card").forEach((el) => {
    el.addEventListener("mouseenter", activateRing);
    el.addEventListener("mouseleave", deactivateRing);
  });

  const statsSection = document.getElementById("stats-trigger");
  if (statsSection) {
    new IntersectionObserver((entries) => entries.forEach((e) => e.isIntersecting && animateCounters()), { threshold: 0.4 }).observe(statsSection);
  }
  const skillsSection = document.getElementById("expertise");
  if (skillsSection) {
    new IntersectionObserver((entries) => entries.forEach((e) => e.isIntersecting && revealBars()), { threshold: 0.3 }).observe(skillsSection);
  }
});

onUnmounted(() => {
  window.removeEventListener("mousemove", onMouseMove);
  window.removeEventListener("scroll", onScroll);
  if (raf) cancelAnimationFrame(raf);
  if (carouselRaf) cancelAnimationFrame(carouselRaf);
});
//contact form submition 
const contactInfo = {
  title: "Let's Build Something Amazing Together",
  description:
    "Have a project idea or need a reliable Full Stack Developer? I'm available for freelance projects, SaaS development, eCommerce platforms, custom web applications, and long-term collaborations. Share your requirements and I'll get back to you as soon as possible.",

  email: "your-email@example.com",
  phone: "+92 XXX XXXXXXX",
  location: "Pakistan",

  availability:
    "Available for freelance projects, remote work, and full-stack development opportunities.",
};
const contactSection = {
  badge: "Get In Touch",

  heading: "Let's Build Something Amazing Together",

  description:
    "Have a project idea or need an experienced Full Stack Developer? I help startups, businesses, and agencies build modern, scalable, and high-performance web applications using Laravel, Vue.js, PHP, MySQL, REST APIs, and modern frontend technologies. Share your requirements and let's discuss how we can turn your idea into a powerful digital solution.",

  contactInfo: [
    {
      icon: "mail",
      title: "Email",
      value: "your-email@example.com",
      link: "mailto:your-email@example.com",
    },
    {
      icon: "phone",
      title: "Phone",
      value: "+92 XXX XXXXXXX",
      link: "tel:+92XXXXXXXXXX",
    },
    {
      icon: "location",
      title: "Location",
      value: "Pakistan",
    },
    {
      icon: "clock",
      title: "Availability",
      value: "Available for Freelance & Remote Projects",
    },
  ],

  expertise: [
    "Full Stack Web Development",
    "Laravel Applications",
    "Vue.js Frontend Development",
    "SaaS Product Development",
    "E-Commerce Platforms",
    "REST API Development",
    "Admin Dashboard Systems",
    "Database Architecture",
  ],

  form: {
    title: "Start Your Project",

    description:
      "Tell me about your project requirements, goals, and timeline. I will review your details and get back to you shortly.",

    fields: [
      {
        name: "name",
        label: "Full Name",
        type: "text",
        placeholder: "John Smith",
        required: true,
      },

      {
        name: "email",
        label: "Email Address",
        type: "email",
        placeholder: "john@example.com",
        required: true,
      },

      {
        name: "phone",
        label: "Phone Number",
        type: "tel",
        placeholder: "+92 300 0000000",
        required: true,
      },

      {
        name: "company",
        label: "Company Name",
        type: "text",
        placeholder: "Your company name",
        required: false,
      },

      {
        name: "service",
        label: "What service do you need?",
        type: "select",
        required: true,

        options: [
          "Full Stack Website Development",
          "Laravel Web Application",
          "Vue.js Application",
          "SaaS Platform Development",
          "E-Commerce Website",
          "API Development & Integration",
          "Admin Dashboard Development",
          "Custom Software Solution",
        ],
      },

      {
        name: "budget",
        label: "Project Budget",
        type: "select",
        required: false,

        options: [
          "$500 - $1000",
          "$1000 - $3000",
          "$3000 - $5000",
          "$5000+",
          "Need Consultation",
        ],
      },

      {
        name: "timeline",
        label: "Project Timeline",
        type: "select",
        required: false,

        options: [
          "Less than 1 month",
          "1 - 3 months",
          "3 - 6 months",
          "Long Term Partnership",
        ],
      },

      {
        name: "message",
        label: "Project Details",
        type: "textarea",
        placeholder:
          "Describe your project idea, required features, technology preferences, and goals...",
        required: true,
      },
    ],

    button: {
      text: "Send Project Inquiry",
      loadingText: "Sending...",
    },

    messages: {
      success:
        "Thank you for contacting me! I will review your project details and respond soon.",

      error:
        "Unable to send your message. Please try again or contact me directly.",
    },
  },


  socialLinks: [
    {
      name: "LinkedIn",
      url: "https://linkedin.com/in/your-profile",
      icon: "linkedin",
    },

    {
      name: "GitHub",
      url: "https://github.com/your-profile",
      icon: "github",
    },

    {
      name: "Upwork",
      url: "https://www.upwork.com/freelancers/~01373aff88b5f1218d?mp_source=share",
      icon: "briefcase",
    },

    {
      name: "Fiverr",
      url: "https://fiverr.com/your-profile",
      icon: "globe",
    },
  ],


  footerText:
    "Open for freelance projects, collaborations, and full-time opportunities.",
};
const downloadResume = () => {



const resumePath = "/resume/Talha-Abbas-Full-Stack-Developer-Resume.pdf";


  const link = document.createElement("a");



  link.href = resumePath;



  link.download = "Talha-Abbas-Full-Stack-Developer-Resume.pdf";



  document.body.appendChild(link);



  link.click();



  document.body.removeChild(link);



};
</script>

<template>
  <div class="custom-scrollbar overflow-x-hidden relative">
    <!-- Custom cursor -->
    <div ref="cursorGlow" class="cursor-glow"></div>
    <div ref="cursorDot" class="cursor-dot"></div>
    <div ref="cursorRing" class="cursor-ring" :class="{ 'is-active': ringActive }"></div>

    <!-- Nav -->
    <nav ref="navEl" class="fixed top-0 w-full z-50 nav-glass border-b border-white/10 transition-transform duration-300">
      <div class="flex justify-between items-center px-gutter max-w-container-max mx-auto h-20">
        <div class="flex items-center gap-2 font-headline-lg text-headline-lg font-bold text-on-surface tracking-tight">
          <span class="w-2.5 h-2.5 rounded-full bg-secondary shadow-[0_0_12px_rgba(78,222,163,0.8)]"></span>
          Talha Abbas 
        </div>
        <div class="hidden md:flex items-center gap-8">
          <a @click.prevent="scrollToId('work')" href="#work" class="font-body-md text-body-md text-on-surface-variant hover:text-secondary transition-colors px-3 py-1 rounded">Work</a>
          <a @click.prevent="scrollToId('expertise')" href="#expertise" class="font-body-md text-body-md text-on-surface-variant hover:text-secondary transition-colors px-3 py-1 rounded">Expertise</a>
          <a @click.prevent="scrollToId('experience')" href="#experience" class="font-body-md text-body-md text-on-surface-variant hover:text-secondary transition-colors px-3 py-1 rounded">Experience</a>
          <a @click.prevent="scrollToId('services')" href="#services" class="font-body-md text-body-md text-on-surface-variant hover:text-secondary transition-colors px-3 py-1 rounded">Services</a>
        </div>
      <button
  @click="downloadResume"
  class="bg-secondary text-on-secondary px-6 py-2.5 rounded-full font-label-md text-label-md hover:opacity-90 hover:shadow-[0_0_20px_rgba(78,222,163,0.4)] transition-all active:scale-90"
>
  Hire Me
</button>
      </div>
    </nav>

    <main class="relative z-10">
      <!-- ============ HERO ============ -->
      <section id="hero" class="relative min-h-screen flex items-center pt-20 hero-gradient overflow-hidden">
        <div class="grid-overlay"></div>
        <div class="max-w-container-max mx-auto px-gutter grid grid-cols-1 md:grid-cols-12 gap-gutter items-center relative">
          <div class="md:col-span-7 z-10">
            <div class="inline-flex items-center gap-2 px-3 py-1 rounded-full border border-outline-variant bg-surface-container-low mb-6">
              <span class="w-2 h-2 rounded-full bg-secondary animate-pulse"></span>
              <span class="font-label-md text-label-md text-secondary">Available for New Projects</span>
            </div>
            <h1 class="font-display-lg text-display-lg-mobile md:text-display-lg text-on-surface mb-6">
              Full Stack <br />
              <span class="text-transparent bg-clip-text bg-gradient-to-r from-secondary via-secondary to-tertiary">Web Developer</span>
            </h1>
            <p class="font-body-lg text-body-lg text-on-surface-variant max-w-xl mb-4">
              I build resilient, high-performance web applications with Laravel, Vue.js, and Flutter — turning complex requirements into elegant, working products.
            </p>
            <p class="font-body-lg text-body-lg text-on-surface-variant max-w-xl mb-10">
              More than code — I help your <span class="text-secondary font-semibold">business grow on digital platforms</span>, from custom SaaS tools to Shopify and WordPress stores that are built to convert and scale.
            </p>
            <div class="flex flex-wrap gap-4">
              <button class="px-8 py-4 bg-gradient-to-r from-secondary-container to-secondary text-on-secondary-container font-label-md text-label-md rounded-xl hover:shadow-lg hover:shadow-secondary/30 transition-all active:scale-95 flex items-center gap-2">
                Hire Me
                <span class="material-symbols-outlined text-sm">arrow_forward</span>
              </button>
              <button @click="scrollToId('work')" class="px-8 py-4 glass-card text-on-surface font-label-md text-label-md rounded-xl hover:bg-white/5 transition-all active:scale-95">
                View Projects
              </button>
              <button  @click="downloadResume" class="px-8 py-4 border border-outline-variant text-on-surface-variant font-label-md text-label-md rounded-xl hover:text-on-surface hover:border-on-surface transition-all flex items-center gap-2">
                <span class="material-symbols-outlined text-sm">download</span>
                Resume
              </button>
            </div>
          </div>

          <!-- Profile photo -->
          <div class="md:col-span-5 relative mt-16 md:mt-0">
            <div class="relative w-full aspect-square flex items-center justify-center">
              <div class="absolute w-[130%] h-[130%] bg-secondary/10 rounded-full blur-[90px]"></div>

              <div class="relative w-72 h-72 md:w-80 md:h-80 rounded-[2rem] p-[3px] photo-ring">
                <div class="w-full h-full rounded-[calc(2rem-3px)] overflow-hidden glass-card">
                  <img src="\images\WhatsApp Image 2026-01-05 at 1.10.05 PM.jpeg" alt="Portrait" class="w-full h-full object-cover" />
                </div>
              </div>

              <div class="absolute -top-4 -right-4 animate-float" style="animation-delay: 0s">
                <div class="glass-card px-5 py-3 rounded-2xl flex items-center gap-3 border-secondary/30">
                  <div class="w-10 h-10 bg-surface-container-highest rounded-lg flex items-center justify-center">
                    <span class="material-symbols-outlined text-secondary">view_quilt</span>
                  </div>

                  <div>
                    <p class="font-label-md text-label-md text-on-surface">Vue.js</p>
                    <p class="text-[10px] text-on-surface-variant">Frontend</p>
                  </div>
                </div>
              </div>

              <div class="absolute top-1/2 -left-10 animate-float" style="animation-delay: 1s">
                <div class="glass-card px-5 py-3 rounded-2xl flex items-center gap-3 border-tertiary/30">
                  <div class="w-10 h-10 bg-surface-container-highest rounded-lg flex items-center justify-center">
                    <span class="material-symbols-outlined text-tertiary">database</span>
                  </div>
                  <div>
                    <p class="font-label-md text-label-md text-on-surface">Laravel</p>
                    <p class="text-[10px] text-on-surface-variant">Backend</p>
                  </div>
                </div>
              </div>

              <div class="absolute bottom-2 right-6 animate-float" style="animation-delay: 2s">
                <div class="glass-card px-5 py-3 rounded-2xl flex items-center gap-3 border-outline/30">
                  <div class="w-10 h-10 bg-surface-container-highest rounded-lg flex items-center justify-center">
                    <span class="material-symbols-outlined text-on-surface">storefront</span>
                  </div>
                  <div>
                    <p class="font-label-md text-label-md text-on-surface">Shopify</p>
                    <p class="text-[10px] text-on-surface-variant">E-Commerce</p>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- ============ EXPERTISE (bento skills) ============ -->
      <section id="expertise" class="py-section-gap-desktop bg-surface-container-lowest px-gutter">
        <div class="max-w-container-max mx-auto">
          <div class="max-w-3xl mb-16">
            <span class="text-secondary font-label-md text-label-md uppercase tracking-widest mb-4 block">Craftsmanship &amp; Code</span>
            <h2 class="font-display-lg text-display-lg-mobile md:text-display-lg text-on-surface mb-6">
              Mastering the <span class="text-gradient">Modern Stack</span>
            </h2>
            <p class="font-body-lg text-body-lg text-on-surface-variant leading-relaxed">
              A multi-disciplinary approach to engineering, bridging high-performance backends, pixel-perfect frontends, and platforms that grow your business.
            </p>
          </div>

          <div class="grid grid-cols-1 md:grid-cols-12 gap-6 mb-20">
            <!-- Frontend -->
            <div class="tilt-card md:col-span-4 glass-panel p-8 rounded-xl" @mousemove="tiltMove" @mouseleave="tiltLeave">
              <div class="flex items-center gap-4 mb-8">
                <div class="w-12 h-12 rounded-lg bg-secondary-container/20 flex items-center justify-center">
                  <span class="material-symbols-outlined text-secondary">fluid_med</span>
                </div>
                <h3 class="font-headline-lg text-headline-lg text-on-surface">Frontend</h3>
              </div>
              <div class="space-y-6">
                <div v-for="s in frontendSkills" :key="s.name">
                  <div class="flex justify-between mb-2">
                    <span class="font-label-md text-label-md text-on-surface-variant">{{ s.name }}</span>
                    <span class="font-label-md text-label-md text-secondary">{{ s.pct }}%</span>
                  </div>
                  <div class="h-1.5 w-full bg-surface-container-highest rounded-full overflow-hidden">
                    <div class="skill-progress-bar h-full bg-secondary" :style="{ width: barsRevealed ? s.pct + '%' : '0%' }"></div>
                  </div>
                </div>
                <div class="flex flex-wrap gap-2 mt-4">
                  <span class="px-3 py-1 rounded-full bg-secondary/10 text-secondary text-[12px] font-semibold border border-secondary/20">Bootstrap</span>
                  <span class="px-3 py-1 rounded-full bg-secondary/10 text-secondary text-[12px] font-semibold border border-secondary/20">Vite</span>
                </div>
              </div>
            </div>

            <!-- Backend -->
            <div class="tilt-card md:col-span-5 glass-panel p-8 rounded-xl" @mousemove="tiltMove" @mouseleave="tiltLeave">
              <div class="flex items-center gap-4 mb-8">
                <div class="w-12 h-12 rounded-lg bg-on-tertiary-container/20 flex items-center justify-center">
                  <span class="material-symbols-outlined text-tertiary">dns</span>
                </div>
                <h3 class="font-headline-lg text-headline-lg text-on-surface">Backend</h3>
              </div>
              <div class="space-y-6">
                <div v-for="s in backendSkills" :key="s.name">
                  <div class="flex justify-between mb-2">
                    <span class="font-label-md text-label-md text-on-surface-variant">{{ s.name }}</span>
                    <span class="font-label-md text-label-md text-tertiary">{{ s.pct }}%</span>
                  </div>
                  <div class="h-1.5 w-full bg-surface-container-highest rounded-full overflow-hidden">
                    <div class="skill-progress-bar h-full bg-tertiary" :style="{ width: barsRevealed ? s.pct + '%' : '0%' }"></div>
                  </div>
                </div>
                <div class="flex flex-wrap gap-2 mt-4">
                  <span class="px-3 py-1 rounded-full bg-tertiary/10 text-tertiary text-[12px] font-semibold border border-tertiary/20">Sanctum</span>
                  <span class="px-3 py-1 rounded-full bg-tertiary/10 text-tertiary text-[12px] font-semibold border border-tertiary/20">MySQL</span>
                </div>
              </div>
            </div>

            <!-- Platforms -->
            <div class="md:col-span-3 space-y-6">
              <div class="tilt-card glass-panel p-6 rounded-xl h-1/2 flex flex-col justify-center" @mousemove="tiltMove" @mouseleave="tiltLeave">
                <div class="flex items-center gap-3 mb-4">
                  <span class="material-symbols-outlined text-on-surface-variant">storefront</span>
                  <h4 class="font-label-md text-label-md text-on-surface">Shopify / Wix</h4>
                </div>
                <p class="text-[13px] text-on-surface-variant">Store builds and custom theming for fast-launch e-commerce.</p>
              </div>
              <div class="tilt-card glass-panel p-6 rounded-xl h-1/2 flex flex-col justify-center" @mousemove="tiltMove" @mouseleave="tiltLeave">
                <div class="flex items-center gap-3 mb-4">
                  <span class="material-symbols-outlined text-on-surface-variant">language</span>
                  <h4 class="font-label-md text-label-md text-on-surface">WordPress</h4>
                </div>
                <p class="text-[13px] text-on-surface-variant">Custom themes, plugins, and performance tuning.</p>
              </div>
            </div>
          </div>

          <!-- 360° Skill Carousel -->
          <div>
            <div class="text-center mb-14">
              <span class="text-secondary font-label-md text-label-md uppercase tracking-widest mb-3 block">The Full Toolkit</span>
              <h3 class="font-headline-xl text-headline-xl text-on-surface">Every skill, one orbit</h3>
            </div>
            <div
              class="carousel-scene"
              @mouseenter="carouselPaused = true"
              @mouseleave="carouselPaused = false"
            >
              <div class="carousel-ring" :style="{ transform: `translateZ(-${CAROUSEL_RADIUS}px) rotateY(${carouselAngle}deg)` }">
                <div
                  v-for="item in carouselItems"
                  :key="item.name"
                  class="carousel-item glass-card"
                  :class="`border-${item.color}/25`"
                  :style="item.style"
                >
                  <span class="material-symbols-outlined text-3xl mb-2" :class="`text-${item.color}`">{{ item.icon }}</span>
                  <span class="font-label-md text-label-md text-on-surface">{{ item.name }}</span>
                </div>
              </div>
              <div class="carousel-floor"></div>
            </div>
            <p class="text-center font-label-md text-label-md text-on-surface-variant mt-8">Hover the orbit to pause · touch and drag on mobile to look around</p>
          </div>
        </div>
      </section>

      <!-- ============ STATS ============ -->
      <section id="stats-trigger" class="pt-section-gap-mobile pb-16 px-gutter max-w-container-max mx-auto">
        <div class="grid grid-cols-1 md:grid-cols-3 gap-gutter">
          <div v-for="stat in stats" :key="stat.label" class="tilt-card glass-card p-8 rounded-xl text-center" @mousemove="tiltMove" @mouseleave="tiltLeave">
            <div class="inline-flex items-center justify-center w-16 h-16 rounded-full mb-4" :class="`bg-${stat.color}/10 text-${stat.color}`">
              <span class="material-symbols-outlined text-4xl">{{ stat.icon }}</span>
            </div>
            <div class="font-display-lg text-display-lg text-on-surface mb-2">{{ stat.value }}{{ stat.label.includes('%') ? '%' : '+' }}</div>
            <div class="font-label-md text-label-md text-on-surface-variant uppercase tracking-widest">{{ stat.label }}</div>
          </div>
        </div>
      </section>

      <!-- ============ SERVICES ============ -->
      <section id="services" class="py-section-gap-desktop px-gutter max-w-container-max mx-auto">
        <div class="mb-16">
          <span class="text-secondary font-label-md text-label-md uppercase tracking-widest mb-3 block">What I Deliver</span>
          <h2 class="font-display-lg text-display-lg text-on-surface mb-4">Core <span class="primary-gradient-text">Services</span></h2>
          <p class="font-body-lg text-body-lg text-on-surface-variant max-w-2xl">
            Precision-engineered software solutions tailored for modern business challenges — from architecture to growth on digital platforms.
          </p>
        </div>
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-gutter">
          <div v-for="svc in services" :key="svc.title" class="tilt-card glass-card p-8 rounded-xl flex flex-col h-full" @mousemove="tiltMove" @mouseleave="tiltLeave">
            <div class="mb-6">
              <span class="material-symbols-outlined text-5xl" :class="`text-${svc.color}`">{{ svc.icon }}</span>
            </div>
            <h3 class="font-headline-lg text-headline-lg text-on-surface mb-4">{{ svc.title }}</h3>
            <p class="font-body-md text-body-md text-on-surface-variant mb-8 flex-grow">{{ svc.desc }}</p>
            <div class="flex flex-wrap gap-2">
              <span v-for="tag in svc.tags" :key="tag" class="px-3 py-1 rounded-full text-[12px] font-semibold" :class="`bg-${svc.color}/10 text-${svc.color}`">{{ tag }}</span>
            </div>
          </div>
        </div>
      </section>

      <!-- ============ EXPERIENCE TIMELINE ============ -->
      <section id="experience" class="py-section-gap-desktop px-gutter bg-surface-container-low/30">
        <div class="max-w-container-max mx-auto">
          <div class="text-center mb-20">
            <h2 class="font-display-lg text-display-lg text-on-surface">The <span class="primary-gradient-text">Trajectory</span></h2>
            <p class="font-body-lg text-body-lg text-on-surface-variant max-w-xl mx-auto mt-4">
              Building across Laravel, Vue.js, and Flutter — one shipped product at a time.
            </p>
          </div>

          <div class="relative max-w-4xl mx-auto">
            <div class="absolute left-0 md:left-1/2 top-0 bottom-0 w-px timeline-line -translate-x-1/2 hidden md:block"></div>

            <div v-for="(item, i) in experience" :key="item.role" class="relative mb-16 md:mb-24 flex flex-col md:flex-row items-center justify-between">
              <div class="w-full md:w-[45%]" :class="i % 2 === 0 ? 'order-2 md:order-1' : 'order-2'">
                <div class="tilt-card glass-card p-8 rounded-xl" @mousemove="tiltMove" @mouseleave="tiltLeave">
                  <div class="font-label-md text-label-md mb-2" :class="`text-${item.color}`">{{ item.year }}</div>
                  <h3 class="font-headline-lg text-headline-lg text-on-surface mb-1">{{ item.role }}</h3>
                  <div class="font-body-md text-body-md mb-4" :class="`text-${item.color}`">{{ item.company }}</div>
                  <ul class="space-y-3 font-body-md text-body-md text-on-surface-variant">
                    <li v-for="p in item.points" :key="p" class="flex items-start gap-2">
                      <span class="material-symbols-outlined text-[18px] mt-1" :class="`text-${item.color}`">check_circle</span>
                      {{ p }}
                    </li>
                  </ul>
                </div>
              </div>
              <div class="absolute left-0 md:left-1/2 w-4 h-4 rounded-full -translate-x-1/2 z-10 hidden md:block" :class="`bg-${item.color}`"></div>
              <div class="w-full md:w-[45%] mb-6 md:mb-0" :class="i % 2 === 0 ? 'order-1 md:order-2' : 'order-1'"></div>
            </div>
          </div>
        </div>
      </section>

      <!-- ============ PORTFOLIO / WORK ============ -->
      <section id="work" class="bg-surface-container-lowest py-section-gap-desktop px-gutter">
        <div class="max-w-container-max mx-auto">
          <div class="flex flex-col md:flex-row justify-between items-end mb-16 gap-8">
            <div class="max-w-2xl">
              <h2 class="font-headline-xl text-headline-xl text-on-surface mb-4">Featured <span class="text-gradient">Case Studies</span></h2>
              <p class="font-body-md text-body-md text-on-surface-variant">Selected projects across SaaS, e-learning, and mobile.</p>
            </div>
          </div>

          <div class="grid grid-cols-1 md:grid-cols-2 gap-10">
            <div v-for="proj in projects" :key="proj.title" class="tilt-card group relative glass-card rounded-2xl p-8" @mousemove="tiltMove" @mouseleave="tiltLeave">
              <span class="px-3 py-1 bg-surface-container-highest/80 backdrop-blur-md rounded-full text-[12px] font-bold text-on-surface uppercase tracking-wider">{{ proj.tag }}</span>
              <div class="space-y-4 mt-6">
                <div class="flex flex-wrap gap-2">
                  <span v-for="t in proj.stack" :key="t" class="text-[11px] font-bold uppercase tracking-widest px-2 py-1 rounded border" :class="`text-${proj.color} bg-${proj.color}/5 border-${proj.color}/10`">{{ t }}</span>
                </div>
                <h3 class="font-headline-lg text-headline-lg text-on-surface transition-colors" :class="`group-hover:text-${proj.color}`">{{ proj.title }}</h3>
                <p class="font-body-md text-body-md text-on-surface-variant">{{ proj.desc }}</p>
                <div class="flex gap-4 pt-2">
                  <a href="#" class="btn-gradient text-on-secondary px-6 py-2.5 rounded-lg font-label-md text-label-md flex items-center gap-2 hover:opacity-90 transition-opacity">
                    Live Demo <span class="material-symbols-outlined text-[18px]">open_in_new</span>
                  </a>
                  <a href="#" class="px-6 py-2.5 rounded-lg border border-outline-variant font-label-md text-label-md text-on-surface hover:bg-white/5 transition-all">GitHub</a>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- ============ CTA ============ -->
      <section class="relative py-section-gap-desktop overflow-hidden">
        <div class="max-w-container-max mx-auto px-gutter text-center">
          <h2 class="font-display-lg text-display-lg text-on-surface mb-8">Ready to Grow Your Business <span class="primary-gradient-text">Digitally?</span></h2>
          <div class="flex flex-col sm:flex-row items-center justify-center gap-6">
            <button class="bg-secondary text-on-secondary px-10 py-4 rounded-full font-label-md text-label-md hover:shadow-[0_0_30px_rgba(78,222,163,0.4)] transition-all transform hover:-translate-y-1">
              Start Your Project
            </button>
            <button class="border border-outline-variant text-on-surface px-10 py-4 rounded-full font-label-md text-label-md hover:bg-white/5 transition-all">View Resume</button>
          </div>
        </div>
      </section>
    </main>
<!-- ============ CONTACT ============ -->
<section id="contact" class="py-section-gap-desktop bg-surface-container-lowest px-gutter relative overflow-hidden">
  <div class="max-w-container-max mx-auto relative z-10">

    <!-- Header -->
    <div class="text-center max-w-3xl mx-auto mb-16">

      <span class="text-secondary font-label-md text-label-md uppercase tracking-widest mb-4 block">
        {{ contactSection.badge }}
      </span>

      <h2 class="font-display-lg text-display-lg-mobile md:text-display-lg text-on-surface mb-6">
        {{ contactSection.heading }}
      </h2>

      <p class="font-body-lg text-body-lg text-on-surface-variant leading-relaxed">
        {{ contactSection.description }}
      </p>

    </div>


    <div class="grid lg:grid-cols-3 gap-10">


      <!-- Left Column -->
      <div class="lg:col-span-1 space-y-8">


        <!-- Contact Information -->
        <div 
          class="tilt-card glass-card p-8 rounded-xl"
          @mousemove="tiltMove"
          @mouseleave="tiltLeave"
        >

          <h3 class="font-headline-lg text-headline-lg text-on-surface mb-6">
            Contact Information
          </h3>


          <div class="space-y-5">

            <div
              v-for="info in contactSection.contactInfo"
              :key="info.title"
              class="flex items-center gap-4"
            >

              <div class="w-12 h-12 rounded-xl bg-secondary/10 flex items-center justify-center shrink-0">

                <span class="material-symbols-outlined text-secondary">
                  {{ info.icon }}
                </span>

              </div>



              <div>

                <h4 class="font-label-md text-label-md text-on-surface-variant uppercase tracking-wide mb-1">
                  {{ info.title }}
                </h4>



                <a
                  v-if="info.link"
                  :href="info.link"
                  class="font-body-md text-body-md text-on-surface hover:text-secondary transition-colors"
                >
                  {{ info.value }}
                </a>


                <p
                  v-else
                  class="font-body-md text-body-md text-on-surface"
                >
                  {{ info.value }}
                </p>


              </div>

            </div>

          </div>

        </div>





        <!-- Expertise -->

        <div 
          class="tilt-card glass-card p-8 rounded-xl"
          @mousemove="tiltMove"
          @mouseleave="tiltLeave"
        >

          <h3 class="font-headline-lg text-headline-lg text-on-surface mb-5">
            Expertise
          </h3>


          <div class="flex flex-wrap gap-3">


            <span
              v-for="skill in contactSection.expertise"
              :key="skill"
              class="px-4 py-2 rounded-full text-[13px] font-semibold bg-tertiary/10 text-tertiary border border-tertiary/20"
            >

              {{ skill }}

            </span>


          </div>


        </div>





        <!-- Social Links -->

       


      </div>






      <!-- Contact Form -->

      <div class="lg:col-span-2">


        <div class="glass-panel rounded-2xl p-8 md:p-10">


          <h3 class="font-headline-xl text-headline-xl text-on-surface mb-3">
            {{ contactSection.form.title }}
          </h3>



          <p class="font-body-md text-body-md text-on-surface-variant mb-8">
            {{ contactSection.form.description }}
          </p>





          <form class="space-y-6" @submit.prevent>



            <!-- Name Email -->

            <div class="grid md:grid-cols-2 gap-6">


              <div
                v-for="field in contactSection.form.fields.slice(0,2)"
                :key="field.name"
              >


                <label class="block mb-2 font-label-md text-label-md text-on-surface-variant">

                  {{ field.label }}

                  <span 
                    v-if="field.required"
                    class="text-secondary"
                  >
                    *
                  </span>

                </label>



                <input
                  :type="field.type"
                  :placeholder="field.placeholder"
                  :required="field.required"
                  class="w-full bg-surface-container border border-outline-variant rounded-xl px-4 py-3 text-on-surface placeholder:text-on-surface-variant/50 focus:outline-none focus:border-secondary focus:ring-1 focus:ring-secondary/40 transition-colors"
                />


              </div>


            </div>





            <!-- Phone Company -->


            <div class="grid md:grid-cols-2 gap-6">


              <div
                v-for="field in contactSection.form.fields.slice(2,4)"
                :key="field.name"
              >


                <label class="block mb-2 font-label-md text-label-md text-on-surface-variant">

                  {{ field.label }}

                  <span 
                    v-if="field.required"
                    class="text-secondary"
                  >
                    *
                  </span>


                </label>



                <input
                  :type="field.type"
                  :placeholder="field.placeholder"
                  :required="field.required"
                  class="w-full bg-surface-container border border-outline-variant rounded-xl px-4 py-3 text-on-surface placeholder:text-on-surface-variant/50 focus:outline-none focus:border-secondary focus:ring-1 focus:ring-secondary/40 transition-colors"
                />


              </div>


            </div>







            <!-- Select Fields -->

            <div class="grid md:grid-cols-3 gap-6">


              <div
                v-for="field in contactSection.form.fields.slice(4,7)"
                :key="field.name"
              >


                <label class="block mb-2 font-label-md text-label-md text-on-surface-variant">

                  {{ field.label }}

                  <span 
                    v-if="field.required"
                    class="text-secondary"
                  >
                    *
                  </span>

                </label>




                <select
                  :required="field.required"
                  class="w-full bg-surface-container border border-outline-variant rounded-xl px-4 py-3 text-on-surface focus:outline-none focus:border-secondary focus:ring-1 focus:ring-secondary/40 transition-colors"
                >

                  <option
                    v-for="opt in field.options"
                    :key="opt"
                  >
                    {{ opt }}
                  </option>


                </select>



              </div>


            </div>







            <!-- Message -->


            <div>


              <label class="block mb-2 font-label-md text-label-md text-on-surface-variant">

                Project Details

                <span class="text-secondary">
                  *
                </span>

              </label>




              <textarea
                rows="6"
                required
                :placeholder="contactSection.form.fields[7].placeholder"
                class="w-full bg-surface-container border border-outline-variant rounded-xl px-4 py-3 text-on-surface placeholder:text-on-surface-variant/50 focus:outline-none focus:border-secondary focus:ring-1 focus:ring-secondary/40 transition-colors resize-none"
              ></textarea>



            </div>







            <!-- Submit Button -->


            <button
              type="submit"
              class="w-full bg-secondary text-on-secondary py-4 rounded-xl font-label-md text-label-md hover:shadow-[0_0_20px_rgba(78,222,163,0.4)] transition-all active:scale-95"
            >

              {{ contactSection.form.button.text }}

            </button>



          </form>



        </div>


      </div>


    </div>


  </div>
</section>
    <!-- ============ FOOTER ============ -->
    <footer class="w-full py-section-gap-desktop bg-surface-container-lowest border-t border-outline-variant px-gutter">

  <div class="max-w-container-max mx-auto">

    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-12 gap-10">


      <!-- Brand Section -->

      <div class="lg:col-span-4">

        <div class="font-headline-xl text-headline-xl font-extrabold text-surface-tint mb-4">
          Talha Abbas
        </div>


        <p class="font-body-md text-body-md text-on-surface-variant max-w-sm leading-relaxed">
          Engineering premium digital experiences with Laravel, Vue.js, and Flutter —
          building scalable applications that help businesses grow.
        </p>



        <!-- Social Links -->

        <div class="flex flex-wrap gap-3 mt-8">


          <a
            v-for="social in contactSection.socialLinks"
            :key="social.name"
            :href="social.url"
            target="_blank"
            rel="noopener"
            class="w-11 h-11 rounded-full glass-card flex items-center justify-center text-on-surface-variant hover:text-secondary hover:border-secondary/30 transition-all"
          >

            <span class="font-label-md text-xs font-bold">
              {{ social.name.slice(0,2) }}
            </span>

          </a>


        </div>


      </div>






      <!-- Navigation -->


      <div class="lg:col-span-2">

        <h4 class="font-label-md text-label-md text-on-surface mb-6 uppercase tracking-wide">
          Links
        </h4>


        <nav class="flex flex-col gap-4">


          <a
            href="#"
            class="font-label-md text-label-md text-on-surface-variant hover:text-secondary transition-colors"
          >
            Github
          </a>


          <a
            href="www.linkedin.com/in/talha-abbas-300762383"
            class="font-label-md text-label-md text-on-surface-variant hover:text-secondary transition-colors"
          >
            LinkedIn
          </a>


          <a
            href="#"
            class="font-label-md text-label-md text-on-surface-variant hover:text-secondary transition-colors"
          >
            Resume
          </a>


          <a
            href="#contact"
            class="font-label-md text-label-md text-on-surface-variant hover:text-secondary transition-colors"
          >
            Contact
          </a>


        </nav>


      </div>








      <!-- Services -->


      <div class="lg:col-span-3">


        <h4 class="font-label-md text-label-md text-on-surface mb-6 uppercase tracking-wide">
          Services
        </h4>


        <nav class="flex flex-col gap-4">


          <span class="font-label-md text-label-md text-on-surface-variant">
            Laravel Development
          </span>


          <span class="font-label-md text-label-md text-on-surface-variant">
            Vue.js Applications
          </span>


          <span class="font-label-md text-label-md text-on-surface-variant">
            SaaS Development
          </span>


          <span class="font-label-md text-label-md text-on-surface-variant">
            API Integration
          </span>


          <span class="font-label-md text-label-md text-on-surface-variant">
            E-Commerce Solutions
          </span>


        </nav>


      </div>








      <!-- Contact CTA -->


      <div class="lg:col-span-3">


        <div class="bg-surface-container p-6 rounded-2xl border border-white/5">

  <p class="font-label-md text-label-md text-on-surface mb-3">
    Let's build something great.
  </p>

  <!-- Email -->
  <a
    href="mailto:talhaabaass@gmail.com"
    class="block font-headline-lg text-headline-lg text-secondary hover:underline break-all"
  >
    talhaabaass@gmail.com
  </a>

  <!-- WhatsApp -->
  <a
    href="https://wa.me/923284334815"
    target="_blank"
    rel="noopener"
    class="mt-4 inline-flex items-center gap-3 text-on-surface hover:text-secondary transition-colors"
  >
    <span class="material-symbols-outlined text-secondary">
      call
    </span>

    <span class="font-body-md text-body-md font-medium">
      WhatsApp: +92 300 1234567
    </span>
  </a>

  <p class="font-body-md text-body-md text-on-surface-variant mt-5">
    Available for freelance projects, SaaS products, and long-term collaborations.
  </p>

</div>


      </div>



    </div>






    <!-- Bottom Footer -->


    <div class="border-t border-outline-variant mt-12 pt-8 flex flex-col md:flex-row justify-between items-center gap-4">


      <p class="font-label-md text-label-md text-on-surface-variant text-center md:text-left">
        © 2026 Talha Abbas. Crafted with precision.
      </p>



      <p class="font-label-md text-label-md text-on-surface-variant">
        Laravel • Vue.js • Flutter • Full Stack Development
      </p>



    </div>


  </div>


</footer>
  </div>
</template>
