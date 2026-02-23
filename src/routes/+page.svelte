<script>
  import { base } from '$app/paths';
  import { onMount, tick } from 'svelte';
  import { browser } from '$app/environment';
  import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';
  // import noreDesktop from '/nore_1.svg?raw';
  // import noreMobile from '/static/nore_mobile.svg?raw';

  // SSR-safe defaults — no window references at module level
  let currentStep = null;
  let isUpdating = false;
  let initialLoad = true;
  let isMobile = false;
  let isTablet = false;
  let svgContainer;
  let svgReady = false;
  let introVisible = true;
  let scrollytellingDone = false;

  // Start with desktop SVG for SSR — swap to mobile in onMount if needed
  //let svgContent = noreDesktop;
  let svgContent = '';

  const FULL_VIEWBOX_DESKTOP = [0, 0, 1020.41, 1287.10];
  const FULL_VIEWBOX_TABLET  = [0, 0, 1000, 1287.10];
  const FULL_VIEWBOX_MOBILE  = [0, 0, 623.63,  1082.83];

  const viewBox = tweened(FULL_VIEWBOX_DESKTOP, {
    duration: 1200,
    easing: cubicOut
  });

  let viewBoxString = '';
  viewBox.subscribe(value => {
    viewBoxString = value.map(v => v.toFixed(2)).join(' ');
  });

  const steps = [
    {
      id: 0,
      title: "The Three Sisters of Southeast Ireland",
      description: "The Three Sisters, the rivers Suir, Barrow and Nore run like blue veins through southeast Ireland, their waters tracing ancient paths across rock, soil, and memory. At Waterford Harbour, they meet in a final embrace: the Nore and Barrow merge at Cheekpoint, flowing into the Suir Estuary before reaching the Celtic Sea. Three distinct but united forces shaping the land and the lives of those who dwell beside them.",
      layers: ['rivers', 'rivers_locator', 'rivers_text', 'rivers_image'],
      viewBox: null
    },
    {
      id: 1,
      title: "Counties",
      description: "Our first focus, the River Nore, begins as a seep and spring-fed tickle on the slopes of the Devil's Bit Mountain in Tipperary. From there, it flows roughly 140 km through Laois and Kilkenny before reaching Waterford. Along its journey, it gathers water from dozens of tributaries - fine capillaries feeding a living system, each contributing flow, sediment, nutrients and story.",
      layers: ['county', 'county_locator', 'county_text', 'county_rivers_text','county_image'],
      viewBox: null
    },
    {
      id: 2,
      title: "Tributaries",
      description: "Within the Nore catchment, tributaries such as the Breagagh, Dinin, Erkina, King's, Goul, Douglas, Nuenna and other rivers join the main channel before the Nore meets the Barrow, and ultimately the Suir, on its way to the sea. To live within a catchment is to live within a network. This map shows the full network of streams and rivers that drain this landscape.",
      layers: ['tributaries', 'tributaries_locator', 'tributaries_text', 'tributaries_rivers_text','tributaries_image'],
      viewBox: null
    },
    {
      id: 3,
      title: "Townlands",
      description: "We tend to think of land through the lenses of townland boundaries and property. A catchment tells a different story: it is all the land that feeds the river. Grassland and crops dominate the lowlands, while woodland and bog persist on higher ground. Where farmland meets stream, rain carries nutrients and soil into the water.",
      layers: ['townlands', 'townlands_locator', 'townlands_text', 'townlands_rivers_text', 'townlands_image'],
      viewBox: null
    },
    {
      id: 4,
      title: "Selected Tributaries",
      description: "Tributaries join like capillaries feeding a main vein: the Dinan, Nuenna, and Arrigle from the west; the Goul, King's River, Douglas and Kilmanagh from the east. Gently carving through limestone plains, valleys, and forests, the Nore River winds past Kilkenny's castle and the old mill towns.",
      layers: ['selected_tributaries', 'selected_tributaries_locator', 'selected_tributaries_text', 'selected_tributaries_rivers_text','selected_tributaries_image'],
      viewBox: null
    },
    {
      id: 5,
      title: "Special Protection Areas (SPA)",
      description: "The Nore supports a rich mosaic of habitats: riverine corridors, wetlands, floodplains, and woodlands. This map shows protected areas across the Nore catchment. Green indicates forest cover; hatched areas mark <span class='sac'>Special Areas of Conservation (SACs) </span> and <span class='spa'>Special Protection Areas (SPAs)</span>—designations that recognise the river's importance for habitats and bird species. These protected zones are strongholds for biodiversity, where conservation efforts are most concentrated.",
      layers: ['sac_sap', 'sac_sap_locator', 'sac_sap_text', 'sac_sap_image'],
      viewBox: null
    },
    {
      id: 6,
      title: "Endangered Species",
      description: "Each dot on this map represents a recorded sighting of a protected or endangered species: <span class='otters'>otters</span>, <span class='lapwings'>lapwings</span>, <span class='curlews'>curlews</span>, <span class='hen'>hen harriers</span>, <span class='pine'>pine martens</span>  and the rare <span class='mussel'>freshwater pearl mussel</span>. These are indicator species—their presence signals clean water and intact habitats. Where they cluster, the river thrives. Where they are absent, something deeper may be amiss.",
      layers: ['endangered_species', 'endangered_species_locator', 'endangered_species_text', 'endangered_species_rivers_text', 'endangered_species_image'],
      viewBox: null
    },
    {
      id: 7,
      title: "Invasive Species",
      description: "Yet the system is under pressure. This map shows where invasive species have been recorded: <span class='knotweed'>Japanese knotweed</span>, <span class='balsam'>Himalayan balsam</span>, <span class='hogweed'>giant hogweed</span>, and <span class='squirrel'>grey squirrel</span>. These species outcompete native wildlife and degrade riparian habitats. In 2019, crayfish plague swept through the Nore, wiping out white-clawed crayfish populations. These losses remind us: when indicator species decline, the ecosystem is speaking. ",
      layers: ['invasive_species', 'invasive_species_locator', 'invasive_species_text', 'invasive_species_rivers_text', 'invasive_species_image'],
      viewBox: null
    },
    {
      id: 8,
      title: "Pollution Levels",
      description: "This map shows where pollution pressure is greatest across the Nore catchment. Based on flow and land use, areas are classified by their potential to impact water quality: <span class='very_high'>very high</span> and <span class='high'>high</span>. Darker zones signal where runoff, nutrients, or contaminants are most likely to reach our rivers. These are the places where action can make the biggest difference. The Nore faces many pressures from <span class='agriculture'>agricultural</span> runoff, habitat fragmentation, and groundwater contamination.",
      layers: ['pollution', 'pollution_locator', 'pollution_text', 'pollution_image'],
      viewBox: {
        desktop: [275, 150, 550, 950],
        tablet:  [275, 130, 550, 950],
        mobile:  [125, 140,  403, 702]
      }
    }
  ];

  const allLayers = [
    'rivers', 'rivers_locator', 'rivers_text', 'rivers_image',
    'county', 'county_locator', 'county_text', 'county_rivers_text', 'county_image',
    'tributaries', 'tributaries_locator', 'tributaries_text', 'tributaries_rivers_text', 'tributaries_image',
    'townlands', 'townlands_locator', 'townlands_text', 'townlands_rivers_text', 'townlands_image',
    'selected_tributaries', 'selected_tributaries_locator', 'selected_tributaries_text', 'selected_tributaries_rivers_text', 'selected_tributaries_image',
    'sac_sap', 'sac_sap_locator', 'sac_sap_text', 'sac_sap_image',
    'endangered_species', 'endangered_species_locator', 'endangered_species_text', 'endangered_species_rivers_text', 'endangered_species_image',
    'invasive_species', 'invasive_species_locator', 'invasive_species_text', 'invasive_species_rivers_text', 'invasive_species_image',
    'pollution', 'pollution_locator', 'pollution_text', 'pollution_image'
  ];

  function getSvgEl() {
    return svgContainer?.querySelector('svg') ?? null;
  }

  function initSvgLayers() {
    const svgEl = getSvgEl();
    if (!svgEl) return;

    // Fix xlink:href → href for inline SVG (mobile Safari requires this)
    svgEl.querySelectorAll('image').forEach(img => {
      const xlinkHref = img.getAttributeNS('http://www.w3.org/1999/xlink', 'href');
      if (xlinkHref && !img.getAttribute('href')) {
        img.setAttribute('href', xlinkHref);
      }
    });

    // Hide all layers — JS will reveal them per step
    allLayers.forEach(layerId => {
      const layer = svgEl.getElementById(layerId);
      if (layer) {
        layer.style.display = 'none';
        layer.style.opacity = '0';
      }
    });

    // Apply correct viewBox immediately
    svgEl.setAttribute('viewBox', viewBoxString);

    svgReady = true;

    setTimeout(() => {
      handleScroll();
      preloadImages(svgEl);

      // ✅ wait for step 0 layer images before fading placeholder
      const step0Layers = steps[0].layers;
      const step0Images = [];
      step0Layers.forEach(layerId => {
        const layer = svgEl.getElementById(layerId);
        if (!layer) return;
        layer.querySelectorAll('image').forEach(img => {
          step0Images.push(img);
        });
      });

      let loaded = 0;
      const total = step0Images.length;

      if (total === 0) {
        document.querySelectorAll('.base-placeholder').forEach(p => {
          p.style.transition = 'opacity 2s ease-in-out';
          p.style.transitionDelay = '0.3s';
          p.style.opacity = '0';
        });
        return;
      }

      step0Images.forEach(img => {
        const href = img.getAttribute('href') ||
                    img.getAttributeNS('http://www.w3.org/1999/xlink', 'href');
        const testImg = new Image();
        testImg.onload = () => {
          loaded++;
          if (loaded === total) {
            document.querySelectorAll('.base-placeholder').forEach(p => {
              p.style.transition = 'opacity 2s ease-in-out';
              p.style.transitionDelay = '0.3s';
              p.style.opacity = '0';
            });
          }
        };
        testImg.onerror = () => { loaded++; };
        testImg.src = href;
      });

    }, 50);
  }

  // Reactive: keep viewBox in sync with tweened store
  $: if (browser && svgReady && viewBoxString) {
    const svgEl = getSvgEl();
    if (svgEl) svgEl.setAttribute('viewBox', viewBoxString);
  }

  // Reactive: handle step viewBox zoom changes
  $: if (browser && svgReady && steps[currentStep]) {
    const step = steps[currentStep];
    const fullViewBox = isMobile ? FULL_VIEWBOX_MOBILE :
                        isTablet ? FULL_VIEWBOX_TABLET :
                        FULL_VIEWBOX_DESKTOP;

    // if (step.viewBox) {
    //   const targetViewBox = isMobile ? step.viewBox.mobile :
    //                         isTablet ? step.viewBox.tablet :
    //                         step.viewBox.desktop;

    if (step.viewBox && !isMobile) {
    const targetViewBox = isTablet ? step.viewBox.tablet :
                          step.viewBox.desktop;

      viewBox.set(fullViewBox, { duration: 0 });
      setTimeout(() => {
        viewBox.set(targetViewBox);

        if (currentStep === 8) {
          const svgEl = getSvgEl();
          if (svgEl) {
            const locator = svgEl.getElementById('pollution_locator');
            if (locator) {
              locator.style.transition = 'opacity 0.6s ease-in-out';
              locator.style.opacity = '0';
              setTimeout(() => { locator.style.display = 'none'; }, 600);
            }
          }
        }
      }, 500);

    } else {
      const duration = initialLoad ? 0 : 1200;
      initialLoad = false;
      viewBox.set(fullViewBox, { duration });
    }
  }

  function showStepLayers(stepIndex) {
    const svgEl = getSvgEl();
    if (!svgEl) return;

    // ✅ mobile: if leaving step 8, restore pollution locator and zoom out
    if (isMobile && stepIndex !== 8) {
      const svgEl = getSvgEl();
      if (svgEl) {
        const locator = svgEl.getElementById('pollution_locator');
        if (locator) {
          locator.style.display = 'block';
          locator.style.opacity = '1';
          locator.style.transition = 'none';
        }
      }
    }

    allLayers.forEach(layerId => {
      const layer = svgEl.getElementById(layerId);
      if (layer) {
        layer.style.opacity = '0';
        layer.style.display = 'none';
      }
    });

    if (!steps[stepIndex]) return;

    const layerTimings = {
      'rivers_text': 500, 'county_text': 500, 'tributaries_text': 500,
      'townlands_text': 500, 'selected_tributaries_text': 500,
      'sac_sap_text': 500, 'endangered_species_text': 500,
      'invasive_species_text': 500, 'pollution_text': 500
    };

    steps[stepIndex].layers.forEach(layerId => {
      const layer = svgEl.getElementById(layerId);
      if (!layer) return;
      const delay = layerTimings[layerId] || 0;
      if (delay > 0) {
        layer.style.display = 'block';
        layer.style.opacity = '0';
        setTimeout(() => {
          layer.style.transition = 'opacity 0.6s ease-in-out';
          layer.style.opacity = '1';
        }, delay);
      } else {
        layer.style.display = 'block';
        layer.style.opacity = '1';
      }

      // ✅ swap low-res placeholder for high-res on demand
      // const lazyImg = layer.querySelector('[data-href]');
      // if (lazyImg) {
      //   lazyImg.setAttribute('href', lazyImg.dataset.href);
      //   lazyImg.removeAttribute('data-href');
      // }

    });

    // ✅ county: delay catchment_county and catchment_county_text
    if (stepIndex === 1) {
      const catchmentCounty = svgEl.getElementById('catchment_county');
      const catchmentCountyText = svgEl.getElementById('catchment_county_text');

      // ✅ hide immediately — no transition, before parent renders
      [catchmentCounty, catchmentCountyText].forEach(el => {
        if (!el) return;
        el.style.transition = 'none';   // ← critical, prevents flash
        el.style.opacity = '0';
      });

      // ✅ then fade in after everything else has loaded
      setTimeout(() => {
        [catchmentCounty, catchmentCountyText].forEach(el => {
          if (!el) return;
          el.style.transition = 'opacity 0.6s ease-in-out';
          el.style.opacity = '1';
        });
      }, 1000);
    }

    // ✅ selected_tributaries: delay King's, Dinin, Nuenna subgroups
    if (stepIndex === 4) {
      ["King's", 'Dinin', 'Nuenna'].forEach(id => {
        const el = svgEl.getElementById(id);
        if (!el) return;
        el.style.opacity = '0';
        setTimeout(() => {
          el.style.transition = 'opacity 0.6s ease-in-out';
          el.style.opacity = '1';
        }, 1000); // ← 1s after parent layer appears
      });
    }

    // ✅ Load this step's image on demand only
    // const imageLayerId = allLayers.find(l => 
    //   l.endsWith('_image') && 
    //   steps[stepIndex].layers[0].startsWith(l.replace('_image', ''))
    // );
    // const imageLayer = imageLayerId ? svgEl.getElementById(imageLayerId) : null;
    // if (imageLayer) {
    //   const lazyImg = imageLayer.querySelector('[data-href]');
    //   if (lazyImg) {
    //     lazyImg.setAttribute('href', lazyImg.dataset.href);
    //     lazyImg.removeAttribute('data-href');
    //   }
    //   // ✅ make image layer visible
    //   imageLayer.style.display = 'block';
    //   imageLayer.style.opacity = '1';
    // }
  }

  function preloadImages(svgEl) {
    svgEl.querySelectorAll('[data-href]').forEach(img => {
      const highResSrc = img.dataset.href;
      if (!highResSrc) return;

      // Clone the element — high-res overlay, starts invisible
      const highResImg = img.cloneNode(true);
      highResImg.setAttribute('href', highResSrc);
      highResImg.removeAttribute('data-href');
      highResImg.style.opacity = '0';

      // Insert high-res clone directly after low-res
      img.parentNode.insertBefore(highResImg, img.nextSibling);

      // Once high-res is loaded, fade it in and fade out low-res
      const tempImg = new Image();
      tempImg.onload = () => {
        highResImg.style.transition = 'opacity 0.6s ease-in-out';
        highResImg.style.opacity = '1';
        setTimeout(() => {
          img.style.transition = 'opacity 0.6s ease-in-out';
          img.style.opacity = '0';
        }, 300);
      };
      tempImg.src = highResSrc;
    });
  }

  // function preloadImages(svgEl) {
  //   svgEl.querySelectorAll('[data-href]').forEach(img => {
  //     const src = img.dataset.href;
  //     if (!src) return;
  //     // Method 1: new Image() warms the cache
  //     const tempImg = new Image();
  //     tempImg.src = src;
  //     // Method 2: also set href directly (works if browser downloads despite display:none)
  //     img.setAttribute('href', src);
  //     img.removeAttribute('data-href');
  //   });
  // }

  // function preloadImages(svgEl) {
  //   svgEl.querySelectorAll('[data-href]').forEach(img => {
  //     const src = img.dataset.href;
  //     if (!src) return;
  //     const tempImg = new Image();
  //     tempImg.src = src; // warms cache only — doesn't touch SVG
  //   });
  // }

  // function preloadImages(svgEl) {
  //   const lazyImages = svgEl.querySelectorAll('[data-href]');
  //   if (lazyImages.length === 0) return;

  //   lazyImages.forEach((img, index) => {
  //     setTimeout(() => {
  //       const src = img.dataset.href;
  //       if (!src) return;
  //       const tempImg = new Image();
  //       tempImg.onload = () => {
  //         img.setAttribute('href', src);
  //         img.style.transition = 'opacity 0.5s ease-in-out';
  //         img.style.opacity = '1';
  //         img.removeAttribute('data-href');
  //       };
  //       tempImg.src = src;
  //     }, index * 200);
  //   });
  // }

  function handleScroll() {
    if (isUpdating) return;
    const stepElements = document.querySelectorAll('.step');
    const viewportMiddle = window.innerHeight / 2;
    let detectedStep = null;

    // stepElements.forEach((el, index) => {
    //   const rect = el.getBoundingClientRect();
    //   if (rect.top < viewportMiddle && rect.bottom > viewportMiddle) {
    //     detectedStep = index;
    //   }
    // });

    stepElements.forEach((el, index) => {
      const rect = el.getBoundingClientRect();
      if (isMobile) {
        // ✅ on mobile: layer changes only when previous step is completely gone off top
           // (rect.top <= 0 && rect.bottom > 0)
        if (rect.top < window.innerHeight * 0.30 && rect.bottom > 0) {
          detectedStep = index;
        }
      } else {
        if (rect.top < viewportMiddle && rect.bottom > viewportMiddle) {
          detectedStep = index;
        }
      }
    });

    // ✅ mobile only: reactive zoom for step 8 based on scroll position
    if (isMobile) {
      const step8El = stepElements[8];
      if (step8El) {
        const rect = step8El.getBoundingClientRect();
        const scrolledInto = -rect.top;
        const threshold = step8El.offsetHeight * 0.25;
        const svgEl = getSvgEl();
        if (svgEl) {
          const locator = svgEl.getElementById('pollution_locator');
          if (scrolledInto > threshold) {
            // zoomed in
            viewBox.set([125, 140, 403, 702]);
            if (locator && locator.style.opacity !== '0') {
              locator.style.transition = 'opacity 0.6s ease-in-out';
              locator.style.opacity = '0';
              setTimeout(() => { locator.style.display = 'none'; }, 600);
            }
          } else if (currentStep === 8 || scrolledInto > 0) {
            // scrolled back above threshold — zoom out
            viewBox.set(FULL_VIEWBOX_MOBILE);
            if (locator && locator.style.opacity === '0') {
              locator.style.display = 'block';
              locator.style.transition = 'opacity 0.6s ease-in-out';
              locator.style.opacity = '1';
            }
          }
        }
      }
    }

    // ✅ detect if we've scrolled past all steps
    const lastStep = stepElements[stepElements.length - 1];
    if (lastStep) {
      const lastRect = lastStep.getBoundingClientRect();
      if (isMobile) {
        // ✅ on mobile wait until last step is fully scrolled off screen
        scrollytellingDone = lastRect.bottom < 0;
      } else {
        scrollytellingDone = lastRect.bottom < window.innerHeight;
      }
    }

    let targetStep;
    if (detectedStep !== null) {
      targetStep = detectedStep;
    } else {
      const scrollY = window.scrollY;
      const docHeight = document.documentElement.scrollHeight;
      const winHeight = window.innerHeight;
      if (currentStep === steps.length - 1 && scrollY > 0) {
        targetStep = steps.length - 1;
      } else if (scrollY + winHeight >= docHeight - 50) {
        targetStep = steps.length - 1;
      } else {
        targetStep = 0;
      }
    }

    if (currentStep !== targetStep) {
      isUpdating = true;
      currentStep = targetStep;
      showStepLayers(targetStep);
      setTimeout(() => { isUpdating = false; }, 100);
    }
  }

  function checkIfMobile() {
    isMobile = window.innerWidth <= 768;
    isTablet = window.innerWidth > 768 && window.innerWidth <= 1024;
    const correctViewBox = isMobile ? FULL_VIEWBOX_MOBILE :
                           isTablet ? FULL_VIEWBOX_TABLET :
                           FULL_VIEWBOX_DESKTOP;
    viewBox.set(correctViewBox, { duration: 0 });
    initialLoad = true;
  }

  onMount(async () => {
    // 1. Detect actual screen size
    checkIfMobile();

    // ✅ Lock vh to initial window height — prevents URL bar shift on mobile
    document.documentElement.style.setProperty('--vh', `${window.innerHeight}px`);

    // 2. Fetch only the SVG we actually need
    const url = isMobile ? `${base}/nore_mobile.svg` : `${base}/nore_1.svg`;
    const raw = await fetch(url).then(r => r.text());
    svgContent = raw.replaceAll('/images/', `${base}/images/`);
    await tick();
    initSvgLayers();

    // 3. Track intro panel visibility
    const introPanel = document.querySelector('.intro-panel');
    const introObserver = new IntersectionObserver((entries) => {
      introVisible = entries[0].isIntersecting;
    }, { threshold: 0 });
    introObserver.observe(introPanel);

    window.addEventListener('scroll', handleScroll, { passive: true });
    window.addEventListener('resize', checkIfMobile, { passive: true });

    return () => {
      window.removeEventListener('scroll', handleScroll);
      window.removeEventListener('resize', checkIfMobile);
      introObserver.disconnect();
    };
  });

  // onMount(async () => {
  //   // 1. Detect actual screen size
  //   checkIfMobile();

  //   // 2. Fetch only the SVG we actually need
  //   const url = isMobile ? '/nore_mobile.svg' : '/nore_1.svg';
  //   svgContent = await fetch(url).then(r => r.text());
  //   await tick();
  //   initSvgLayers();

  //   window.addEventListener('scroll', handleScroll, { passive: true });
  //   window.addEventListener('resize', checkIfMobile, { passive: true });

  //   return () => {
  //     window.removeEventListener('scroll', handleScroll);
  //     window.removeEventListener('resize', checkIfMobile);
  //   };
  // });
</script>

<svelte:head>
  <title>Rivers, Conservation & Environmental Impact</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=PT+Serif:wght@400;700&display=swap" rel="stylesheet">
  <link href="https://fonts.googleapis.com/css2?family=Space+Mono&display=swap" rel="stylesheet">
</svelte:head>

<div class="scrollytelling-container">
  <div class="map-container" class:fading={scrollytellingDone}>
    <div class="svg-wrapper">
      <div bind:this={svgContainer} class="svg-container">
        <img src="/images/nore_1-1-low.webp" class="base-placeholder desktop-placeholder" alt="" />
        <img src="/images/nore_mobile-1-low.webp" class="base-placeholder mobile-placeholder" alt="" />
        {@html svgContent}
      </div>
      <div class="edge-blur-overlay desktop-blur"></div>
      <div class="edge-blur-overlay mobile-blur"></div>
    </div>
  </div>

  <div class="scroll-container">
    <!-- INTRO PANEL — appears before step 0 -->
    <div class="intro-panel">
      <div class="info-panel intro-info-panel">
        <h1>The Three Sisters of <br> Southeast Ireland <br></h1>
        <p>A Living Catchment Story <br>This is not a finished story.<br>It is a beginning.<br>
           What you are about to scroll through is a scaffold — a structure to help us see, sense,
          and speak about the Three Sisters catchment of southeast Ireland: the Nore, the Suir, and the Barrow,
          and the lands and lives they connect.
          <br> This story is a primer — for curiosity, for care, and for citizen science.
              It offers shared reference points: rivers and tributaries, soils and species,
              pressures and protections. But it does not pretend to hold the whole truth.<br>
          <br>

        </p>
        <p class="keep-scrolling"> Keep Scrolling ↓</p>
      </div>
    </div>

    {#each steps as step, index}
      <div class="step" class:active={currentStep === index} class:hidden={introVisible}>
        <div class="info-panel">
          <!-- <h2>{step.title}</h2> -->
          <p>{@html step.description}</p>
          <p class="keep-scrolling"> Keep Scrolling ↓</p>
        </div>
      </div>
    {/each}

    <div class="end-spacer"></div>
  </div>
</div>

<div class="ending-section">
  <div class="ending-content" class:visible={scrollytellingDone}>
    <p>Yet this is also a story of care. Local groups like the Nore River Catchment Trust bring farmers, scientists, and anglers together. School children count invertebrates; farmers plant riparian buffers; volunteers test water. Slowly, the invisible networks become visible again. The catchment becomes a shared canvas for stewardship—where every stream has a name, and every stretch of river tells a story.</p>
    <p>An Invitation to Co-Author</p>
    <p>If catchments are circulatory systems, <br>then stories are how they learn to speak.<br></p>
    <p>This publication does not conclude the story of the Nore, the Suir, or the Barrow. It simply creates a common ground from which many stories can grow. The deepest knowledge of a catchment does not live in reports or maps alone, but in fields and kitchens, on riverbanks and bridges, in daily walks and floods remembered decades later.</p>
    <p class="source">Source: EPA Ireland | National Biodiversity Data Centre | 
      National Parks & Wildlife Service Copernicus Sentinel 30m | OpenStreetMap | 
      CORINE Land Cover, Copernicus Land Monitoring Service
    </p>
  </div>
</div>

<style>
  :global(#rivers), :global(#rivers_locator), :global(#rivers_text), :global(#rivers_image),
  :global(#county), :global(#county_locator), :global(#county_text), :global(#county_rivers_text), :global(#county_image),
  :global(#tributaries), :global(#tributaries_locator), :global(#tributaries_text), :global(#tributaries_rivers_text), :global(#tributaries_image),
  :global(#townlands), :global(#townlands_locator), :global(#townlands_text), :global(#townlands_rivers_text), :global(#townlands_image),
  :global(#selected_tributaries), :global(#selected_tributaries_locator), :global(#selected_tributaries_text), :global(#selected_tributaries_rivers_text), :global(#selected_tributaries_image),
  :global(#sac_sap), :global(#sac_sap_locator), :global(#sac_sap_text), :global(#sac_sap_image),
  :global(#endangered_species), :global(#endangered_species_locator), :global(#endangered_species_text), :global(#endangered_species_rivers_text), :global(#endangered_species_image),
  :global(#invasive_species), :global(#invasive_species_locator), :global(#invasive_species_text), :global(#invasive_species_rivers_text), :global(#invasive_species_image),
  :global(#pollution), :global(#pollution_locator), :global(#pollution_text), :global(#pollution_image) {
    display: none;
  }

  :global(body) {
    overflow-y: scroll;
    margin: 0;
    padding: 0;
  }

  .svg-container {
    position: relative;
    width: 100%;
    height: 100%;
    display: block;
  }

  .scrollytelling-container {
    width: 100%;
    min-height: 100vh;
    display: flex;
    position: relative;
    background: #ffffff;
  }

  .scroll-container {
    width: 59.8%;
    margin-right: 40.2%;
    min-height: 100vh;
    position: relative;
    z-index: 2;
    order: 1;
  }

  .map-container {
    position: fixed;
    top: 0;
    right: 0;
    width: 40.2%;
    height: 100vh;
    height: 100dvh;
    overflow: hidden;
    background: #ffffff;
    z-index: 1;
    transition: opacity 0.6s ease-out;
  }

  .map-container.fading {
    opacity: 0;
    pointer-events: none;
  }

  .map-container::before {
    content: '';
    position: absolute;
    left: 0; top: 0; bottom: 0;
    width: 100px;
    z-index: 10;
    pointer-events: none;
  }

  .map-container .svg-container :global(svg) {
    width: 100%;
    height: 100%;
    display: block;
  }

  .step {
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 2rem;
    box-sizing: border-box;
  }

  .step:nth-child(2) {
    align-items: flex-end;
    padding-bottom: 3rem;
  }

  .info-panel {
    background: rgba(255, 255, 255, 0.98);
    padding: 2.5rem;
    border-radius: 16px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
    max-width: 600px;
    width: 100%;
    opacity: 0.5;
    margin-top: 0px;
    transform: translateY(40px);
    transition: opacity 0.4s cubic-bezier(0.4, 0, 0.2, 1), transform 0.4s cubic-bezier(0.4, 0, 0.2, 1), box-shadow 0.4s cubic-bezier(0.4, 0, 0.2, 1);
    font-family: PT Serif;
  }

  .step.active:not(.hidden) .info-panel {
    opacity: 1;
    transform: translateY(0px);
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
  }

  .step.hidden .info-panel {
    opacity: 0 !important;
    pointer-events: none;
    transform: translateY(40px);
    transition: none;
  }

  .info-panel p {
    margin: 0 0 1.5rem 0;
    color: #000000;
    line-height: 1.5;
    font-size: 1.2rem;
  }

  .end-spacer { height: 10vh; }

  .svg-wrapper {
    position: relative;
    width: 100%;
    height: 100%;
  }

  .svg-wrapper .svg-container :global(svg) {
    width: 100%;
    height: 100%;
    display: block;
  }

  .edge-blur-overlay {
    position: absolute;
    top: 0; left: 0;
    width: 100%;
    height: 100%;
    z-index: 10;
    pointer-events: none;
    background-size: cover;
    background-position: left center;
    background-repeat: no-repeat;
  }

  .desktop-blur { background-image: url('/nore_blur_desktop.webp'); }
  .mobile-blur  { display: none; background-image: url('/nore_blur_mobile.webp'); }

  .base-placeholder {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 0;
  }

  .desktop-placeholder {
    display: block;
    object-fit: contain;
    object-position: center top;
    background: #ffffff;
  }

  .mobile-placeholder {
    display: none;
    object-fit: contain;
    object-position: center top;
    background: #ffffff;
  }

  :global(.sac) {
    background-color: #ba957d;
    color: #ffffff;
    padding: 0px 2px;
    border-radius: 3px;
  }

  :global(.spa) {
    background-color: #7a726c;
    color: #ffffff;
    padding: 0px 2px;
    border-radius: 3px;
  }

  :global(.otters) {
    background-color: rgba(224, 187, 228, 0.502);
    color: #232323;
    padding: 0px 2px;
    border-radius: 3px;
    border: 1.5px solid #9c76a0;
  }

  :global(.curlews) {
    background-color: rgba(255, 179, 186, 0.502);
    color: #232323;
    padding: 0px 2px;
    border-radius: 3px;
    border: 1.5px solid #b69296;
  }

  :global(.lapwings) {
    background-color: rgba(255, 255, 186, 0.502);
    color: #232323;
    padding: 0px 2px;
    border-radius: 3px;
    border: 1.5px solid #baba8a;
  }

  :global(.hen) {
    background-color: rgba(186, 225, 255, 0.502);
    color: #232323;
    padding: 0px 2px;
    border-radius: 3px;
    border: 1.5px solid #8ca9bf;
  }

  :global(.pine) {
    background-color: rgba(255, 218, 185, 0.502);
    color: #232323;
    padding: 0px 2px;
    border-radius: 3px;
    border: 1.5px solid #ad957f;
  }

  :global(.mussel) {
    background-color: rgba(186, 255, 201, 0.502);
    color: #232323;
    padding: 0px 2px;
    border-radius: 3px;
    border: 1.5px solid #8bb695;
  }

  :global(.knotweed) {
    background-color: rgba(213, 158, 255, 0.502);
    color: #232323;
    padding: 0px 2px;
    border-radius: 3px;
    border: 1.5px solid #7538a4;
  }

  :global(.balsam) {
    background-color: rgba(126, 155, 88, 0.502);
    color: #232323;
    padding: 0px 2px;
    border-radius: 3px;
    border: 1.5px solid #47603f;
  }

  :global(.hogweed) {
    background-color: rgba(255, 138, 138, 0.502);
    color: #232323;
    padding: 0px 2px;
    border-radius: 3px;
    border: 1.5px solid #762c2c;
  }

  :global(.squirrel) {
    background-color: rgba(247, 127, 0, 0.502);
    color: #232323;
    padding: 0px 2px;
    border-radius: 3px;
    border: 1.5px solid #a45400;
  }

  :global(.very_high) {
    background-color: #695a55;
    color: #ffffff;
    padding: 0px 2px;
    border-radius: 3px;
  }

  :global(.high) {
    background-color: #e77148;
    color: #ffffff;
    padding: 0px 2px;
    border-radius: 3px;
  }

  :global(.agriculture) {
    background-color: #dae8bb;
    color: #333333;
    padding: 0px 2px;
    border-radius: 3px;
  }

  .intro-panel {
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 2rem;
    box-sizing: border-box;
  }

  .intro-info-panel {
    opacity: 1 !important;
    transform: none !important;
    background: transparent;
    box-shadow: none;
    border-radius: 0;
  }

  .intro-info-panel h1 {
    margin: 0 0 2rem 0;
    color: #1a202c;
    font-size: 2.4rem;
    font-weight: 700;
    line-height: 1.10;
    font-family: PT Serif;
    text-align: center;
  }

  .intro-info-panel p {
    margin: 0;
    color: #232323;
    line-height: 1.5;
    font-size: 1.1rem;
    font-family: PT Serif;
  }

  .ending-section {
    position: relative;
    z-index: 20;
    background: #ffffff;
    min-height: 100vh;
    padding: 80px 40px;
    box-sizing: border-box;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .ending-content {
    max-width: 600px;
    font-family: PT Serif;
  }

  .ending-content p {
    font-size: 1.25rem;
    line-height: 1.6;
    color: #232323;
    margin: 0 0 1.5rem 0;
  }

  .source {
    font-size: 0.85rem !important;
    color: #353535 !important;
    font-family: 'Space Mono', monospace !important;
    line-height: 1rem !important;
  }

  :global(.keep-scrolling) {
    text-align: center;
    color: #353535;
    font-size: 1.25rem !important;
    font-family: 'Space Mono', monospace !important;
    margin: 1rem 0 0 0;
    opacity: 0.8;
    letter-spacing: 0.02em;
  }

  .intro-info-panel :global(.keep-scrolling) {
    margin-top: 3rem;
  }

  /* ===== TABLET ===== */
  @media (max-width: 1024px) and (min-width: 769px) {
    .scrollytelling-container { flex-direction: column; height: auto; }

    .map-container {
      position: fixed;
      top: 0; left: 0;
      width: 100%;
      height: 100vh;
      height: 100dvh;
      z-index: 1;
      padding: 0; margin: 0;
      display: block;
      background: #ffffff;
      overflow: hidden;
    }

    .map-container .svg-container {
      will-change: transform;
      -webkit-transform: translateZ(0);
      transform: translateZ(0);
    }

    .map-container .svg-container :global(svg) {
      width: 100%;
      height: 100%;
      display: block;
      transform: translateY(-36px);
    }

    .scroll-container {
      flex: none; width: 100%;
      position: relative; z-index: 2;
      order: 2; background: transparent;
    }

    .step {
      padding: 1rem;
      min-height: 100vh;
      align-items: center;
      justify-content: center;
      box-sizing: border-box;
    }

    .info-panel {
      box-sizing: border-box;
      max-width: 100%;
      width: calc(100% - 2rem);
      padding: 2rem 1.5rem;
      background: rgba(242, 242, 242, 0.5);
      backdrop-filter: blur(10px);
      -webkit-backdrop-filter: blur(20px);
      margin: 0 1rem;
      border-radius: 12px;
      transform: translateY(20px);
    }

    .info-panel p {
      font-size: 1.5rem !important;
      margin: 0 0 1.5rem 0;
      color: #000000;
    }

    .step.active:not(.hidden) .info-panel {
      opacity: 0.90;
      transform: translateY(0px);
    }

    .mobile-placeholder { display: none; }

    .desktop-placeholder {
      display: block;
      object-fit: contain;
      object-position: center top;
      background: #ffffff;
    }

    .desktop-blur { display: none; }
    .mobile-blur  { display: block; }

    .intro-panel {
      box-sizing: border-box;
      align-items: center;
      padding-bottom: 0;
    }

    .intro-info-panel {
      box-sizing: border-box;
      background: rgba(230, 230, 230, 0.25) !important;
      backdrop-filter: blur(10px);
      -webkit-backdrop-filter: blur(10px);
      max-width: 100%;
      width: calc(100% - 2rem);
      margin: 0 1rem;
    }

    .ending-section {
      padding: 60px 2rem;
      align-items: center;
    }

    .ending-content p {
      font-size: 2rem;
      line-height: 1.3;
    }

    .source {
      font-size: 1rem !important;
      color: #353535 !important;
      font-family: 'Space Mono', monospace !important;
      line-height: 1.2rem !important;
    }
  }

  /* ===== MOBILE ===== */
  @media (max-width: 768px) {
    .scrollytelling-container { flex-direction: column; height: auto; }

    .map-container {
      position: fixed;
      top: 0; left: 0;
      width: 100%;
      /* height: 100vh;
      height: 100dvh; */
      height: var(--vh, 100dvh);  /* ← was: 100vh / 100dvh */
      height: var(--vh, 100dvh);
      z-index: 1;
      padding: 0; margin: 0;
      display: block;
      background: #ffffff;
      overflow: hidden;
    }

    .map-container .svg-container {
      width: 100%;
      height: 100%;
    }

    .map-container .svg-container :global(svg) {
      width: 100%;
      height: 100%;
      display: block;
      position: absolute;
      top: 0;
      left: 0;
    }

    .scroll-container {
      flex: none; width: 100%;
      position: relative; z-index: 2;
      order: 2; background: transparent;
    }

    .step {
      padding: 1rem;
      min-height: 100vh;
      align-items: center;
      justify-content: center;
      box-sizing: border-box;
    }

    .info-panel {
      box-sizing: border-box;
      max-width: 100%;
      width: calc(100% - 2rem);
      padding: 2rem 1.5rem;
      background: rgba(242, 242, 242, 0.5);
      backdrop-filter: blur(10px);
      -webkit-backdrop-filter: blur(20px);
      margin: 0 1rem;
      border-radius: 12px;
      box-shadow: 0 4px 24px rgba(0,0,0,0.06), 0 2px 8px rgba(0,0,0,0.04);
      opacity: 0.85;
      transform: translateY(20px);
    }

    .step.active:not(.hidden) .info-panel {
      opacity: 0.90;
      transform: translateY(0px);
      box-shadow: 0 8px 24px rgba(0,0,0,0.12);
    }

    .info-panel p {
      font-size: 0.75rem !important;
      margin: 0 0 0.25rem 0;
      line-height: 1.25rem !important;
      color: #000000;
    }

    .end-spacer { height: 150vh; }
    .desktop-blur { display: none; }
    .mobile-blur  { display: block; }

    .mobile-placeholder {
      display: block;
      object-fit: contain;
      object-position: center top;
      background: #ffffff;
    }

    .desktop-placeholder { display: none; }

    .intro-panel {
      box-sizing: border-box;
      min-height: 100vh;
      align-items: flex-end;
      padding-bottom: 6rem;
    }

    .intro-info-panel {
      box-sizing: border-box;
      background: rgba(230, 230, 230, 0.25) !important;
      backdrop-filter: blur(10px);
      -webkit-backdrop-filter: blur(10px);
      width: calc(100% - 2rem);
      margin: 0 1rem;
    }

    .intro-info-panel h1 {
      font-size: 1.40rem !important;
      color: rgb(0, 0, 0);
      line-height: 1.20;
      margin: 0 0 0.75rem 0;
    }

    .intro-info-panel p {
      font-size: 0.75rem !important;
      line-height: 1.15 !important;
      margin: 0;
    }

    .intro-info-panel :global(.keep-scrolling) {
      margin-top: 0.5rem;
    }

    .ending-section {
      padding: 40px 1.5rem;
      align-items: flex-start;
      box-sizing: border-box;
    }

    .ending-content {
      max-width: 100%;
      box-sizing: border-box;
    }

    .ending-content p {
      font-size: 1.2rem;
      line-height: 1.6;
    }

    .source {
      font-size: 0.8rem !important;
      color: #353535 !important;
      font-family: 'Space Mono', monospace !important;
      line-height: 0.80rem !important;
    }

    .intro-info-panel :global(.keep-scrolling) {
      font-size: 0.75rem !important;
      margin-top: 0.5rem;
    }
  }

  /* ===== SMALL MOBILE ===== */
  @media (max-width: 480px) {
    .info-panel { padding: 1.5rem 1rem; width: calc(100% - 1rem); margin: 0 0.5rem; }
    .info-panel p { font-size: 0.95rem; }
  }
</style>