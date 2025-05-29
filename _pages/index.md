---
layout: home
title: "Welcome to Dr. Dot's International Tutoring!"
permalink: /
header:
    overlay_image: /assets/images/cover.png
    actions:
        - label: "Contact Me"
          url: /contact/

---

# Welcome

Welcome to my corner of the internet! I'm Dr. Dorothy MacDonald — but my students just call me **Dr. Dot**.
{: .text-justify}

With a Ph.D. in Immunology, 17 years of teaching experience, and a deep love for science, I'm here to help young learners **crack the code of Biology, Chemistry, and all things sciencey** in a way that's actually fun.
{: .text-justify}

## What I Teach
![image-center](/assets/images/what-i-teach-animated.png){: .align-center}

Whether you’re prepping for an exam or just need someone to make mitochondria *a little less boring*, I’ve got you covered:

- <i class="far fa-fw fa-dna"></i> **Biology** – GCSE/N5, Higher, Advanced Higher, IB, A-Level
- <i class="far fa-fw fa-person-running"></i> **Human Biology & Health Science** – including IB SEHS
- <i class="far fa-fw fa-vial-virus"></i> **Chemistry & Physics** – up to GCSE/N5 level
- <i class="far fa-fw fa-brain"></i> **Study Coaching** – coursework, projects, and confidence boosts

## Why Students Love Learning with Me
![image-center](/assets/images/why-students.png){: .align-center}
- I'm a **real scientist** (Ph.D. in Immunology — yep, I’m that into cells!)
- I’ve taught in top schools and colleges across the UK
- I make learning feel **doable, supportive, and even fun**
- I work with students **around the world** — online and flexible
- I believe in every student’s ability to **shine bright in science**

## What People Say

<!-- Render testimonial data as hidden JSON -->
<script type="application/json" id="testimonial-data">
  {{ site.data.testimonials.testimonials | jsonify }}
</script>

<!-- Container to inject the testimonials -->
<div id="testimonial-container"></div>

<script>
  (function () {
    const rawData = document.getElementById('testimonial-data').textContent;
    const testimonials = JSON.parse(rawData);

    // Fisher-Yates shuffle
    function shuffle(array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
      return array;
    }

    const selected = shuffle(testimonials).slice(0, 2);
    const container = document.getElementById('testimonial-container');

    selected.forEach(t => {
      const html = `
        <div class="testimonial">
          <blockquote>
            "${t.content}"<br/>
            &mdash; ${t.name}
</blockquote>
        </div>`;
      container.insertAdjacentHTML('beforeend', html);
    });
  })();
</script>

## Let’s Chat!

Want to know more or book your first session? I’d love to hear from you!

- <i class='far fa-fw fa-envelope'></i> **Email**: [dot@macsix.io](mailto:dot@macsix.io)
- <i class='fab fa-fw fa-whatsapp'></i> **WhatsApp**: [+44 7973 151425](https://wa.me/447973151425)
- <i class='far fa-fw fa-phone'></i> **Phone**: [+44 7973 151425](tel:+447973151425)
- <i class='far fa-fw fa-calendar-days'></i> **Available**: Weekday evenings & weekends in your timezone.
- <i class="far fa-fw fa-map-location-dot"></i> **Location**: Anywhere! I tutor online
- <i class="far fa-fw fa-money-bill"></i> **Rates**: From £40/hour – ask me about bundles & sibling discounts!
