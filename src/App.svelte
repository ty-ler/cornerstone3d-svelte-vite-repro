<script lang="ts">
  import logo from "./assets/svelte.png";
  import Counter from "./lib/Counter.svelte";

  import * as cornerstoneTools from "@cornerstonejs/tools";
  import * as cornerstoneStreamingImageVolumeLoader from "@cornerstonejs/streaming-image-volume-loader";

  import {
    RenderingEngine,
    ImageVolume,
    StackViewport,
    VolumeViewport,
    Viewport,
    Types,
    Enums,
  } from '@cornerstonejs/core';

  console.log(cornerstoneStreamingImageVolumeLoader)
  console.log(ImageVolume);
  console.log(VolumeViewport)
  console.log(Viewport)
  // console.log(RenderingEngine)
  console.log(StackViewport);
  debugger;

  import dicomParser from 'dicom-parser';
  import * as cornerstone from '@cornerstonejs/core';
  import cornerstoneWADOImageLoader from 'cornerstone-wado-image-loader';

  function initCornerstoneWADOImageLoader() {
    cornerstoneWADOImageLoader.external.cornerstone = cornerstone;
    cornerstoneWADOImageLoader.external.dicomParser = dicomParser;
    cornerstoneWADOImageLoader.configure({
      useWebWorkers: true,
      decodeConfig: {
        convertFloatPixelDataToInt: false,
      },
    });

    let maxWebWorkers = 1;

    if (navigator.hardwareConcurrency) {
      maxWebWorkers = Math.min(navigator.hardwareConcurrency, 7);
    }

    var config = {
      maxWebWorkers,
      startWebWorkersOnDemand: false,
      taskConfiguration: {
        decodeTask: {
          initializeCodecsOnStartup: false,
          strict: false,
        },
      },
    };

    cornerstoneWADOImageLoader.webWorkerManager.initialize(config);
  }

  import { init as csRenderInit } from '@cornerstonejs/core';

  // This is for debugging purposes
  console.warn(
    'Click on index.ts to open source code for this example --------->'
  );

  const { ViewportType } = Enums;

  const content = document.getElementById('content');
  const element = document.createElement('div');
  element.id = 'cornerstone-element';
  element.style.width = '500px';
  element.style.height = '500px';

  content.appendChild(element);
  // ============================= //

  const studyUID = '1.3.6.1.4.1.25403.345050719074.3824.20170125112931.11';
  const contentType = 'application%2Fdicom';
  const wadoURIRoot = 'https://server.dcmjs.org/dcm4chee-arc/aets/DCM4CHEE/wado';

  const ctSeriesUID = '1.3.6.1.4.1.25403.345050719074.3824.20170125113028.6';
  const ctObjectUID = '1.3.6.1.4.1.25403.345050719074.3824.20170125113100.3';

  const ptSeriesUID = '1.3.6.1.4.1.25403.345050719074.3824.20170125112950.1';
  const ptObjectUID = '1.3.6.1.4.1.25403.345050719074.3824.20170125112959.5';

  // Instantiate a rendering engine
  const renderingEngineId = 'myRenderingEngine';
  const viewportId = 'CT_STACK';

  const createWADOURIImageId = (params) => {
    return `wadouri:${params.wadoURIRoot}?requestType=WADO&studyUID=${params.studyUID}&seriesUID=${params.seriesUID}&objectUID=${params.objectUID}&contentType=${params.contentType}`;
  };

  const ctImageId = createWADOURIImageId({
    wadoURIRoot,
    studyUID,
    seriesUID: ctSeriesUID,
    objectUID: ctObjectUID,
    contentType,
  });

  const ptImageId = createWADOURIImageId({
    wadoURIRoot,
    studyUID,
    seriesUID: ptSeriesUID,
    objectUID: ptObjectUID,
    contentType,
  });

  /**
  * Runs the demo
  */
  async function run() {
    // Init Cornerstone and related libraries
    initCornerstoneWADOImageLoader();
    await csRenderInit();

    const renderingEngine = new RenderingEngine(renderingEngineId);

    // Create a stack viewport
    const viewportInput = {
      viewportId,
      type: ViewportType.STACK,
      element,
      defaultOptions: {
        background: <Types.Point3>[0.2, 0, 0.2],
      },
    };

    renderingEngine.enableElement(viewportInput);

    // Get the stack viewport that was created
    const viewport = <Types.IStackViewport>(
      renderingEngine.getViewport(viewportId)
    );

    // Define a stack containing a single image
    const stack = [ctImageId];

    // Set the stack on the viewport
    await viewport.setStack(stack);

    // Render the image
    viewport.render();
  }

  run();

</script>

<main>
  <img src={logo} alt="Svelte Logo" />
  <h1>Hello Typescript!</h1>

  <Counter />

  <p>
    Visit <a href="https://svelte.dev">svelte.dev</a> to learn how to build Svelte
    apps.
  </p>

  <p>
    Check out <a href="https://github.com/sveltejs/kit#readme">SvelteKit</a> for
    the officially supported framework, also powered by Vite!
  </p>
</main>

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
  }

  main {
    text-align: center;
    padding: 1em;
    margin: 0 auto;
  }

  img {
    height: 16rem;
    width: 16rem;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4rem;
    font-weight: 100;
    line-height: 1.1;
    margin: 2rem auto;
    max-width: 14rem;
  }

  p {
    max-width: 14rem;
    margin: 1rem auto;
    line-height: 1.35;
  }

  @media (min-width: 480px) {
    h1 {
      max-width: none;
    }

    p {
      max-width: none;
    }
  }
</style>
