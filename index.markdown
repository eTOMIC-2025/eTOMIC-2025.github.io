---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---

<head>
    <link rel="stylesheet" href="styles.css">
    <script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
    <script src="https://unpkg.com/wavesurfer.js@7"></script>
    <script src="https://unpkg.com/wavesurfer.js@7/dist/plugins/regions.min.js"></script>
    <script src="https://unpkg.com/wavesurfer.js@7/dist/plugins/hover.min.js"></script>
    <script src="https://cdn.jsdelivr.net/combine/npm/tone@14.7.58,npm/@magenta/music@1.23.1/es6/core.js,npm/focus-visible@5,npm/html-midi-player@1.5.0"></script>
    <script src="{{ '/assets/js/audio_playback.js' | relative_url }}"></script>
</head>

This is the demo page for the paper: _eTOMIC: Transforming and Organizing Music Ideas to Multi-Track Electronic 
Music Compositions with Full-Song Structure_. We propose the TOMIC (Transforming and Organizing Music Ideas to
Composition) paradigm for music data representation. 
It models a musical piece as a sparse, four-dimensional space defined by **clips** (short audio or MIDI segments), 
**sections** (temporal positions), **tracks** (instrument layers), and **transformations** (elaboration methods). 
Based on this, we propose $$e\text{TOMIC}$$, the first electronic music generation system that adapts the TOMIC data structure 
to produce long-term, multi-track compositions with both MIDI and audio clips, while achieving **robust structural consistency** and **high music quality**. We integrate a 
pre-trained large language model via **in-context learning** with the TOMIC data structure to generate high level 
full-song orchestration, then retrieve clip contents from local sample databases. Moreover, we integrate 
$$e\text{TOMIC}$$ with the REAPER digital audio workstation (DAW) to provide an interactive workflow and exports audio of high 
resolution. In this page, we demonstrate the TOMIC data structure with examples, then we show multiple generated audio demos and a video demo of REAPER DAW integration.

<div class="center-stuff"><img src="/assets/pics/tomic_structure.jpg" style="width:50%" alt=""></div>

---
## TOMIC Data Structure
In this part, we show the examples of applying transformations on raw clips in an **8-bar section** with 120BPM and 4/4 time signature. For the _**action sequence**_ 
attribute of **General Clip** transformations and **Drum** transformations, we use "►" to denote the _start_ state, "=" to 
denote the _sustain_ state, and "-" to denote the _rest_ state. Each state corresponds to an action 
at a step time within the section (eg. a bar has 16 steps in the 4/4 time signature). The _start_ state means the clip will 
be replayed at this time, _rest_ means the clip will stop playing, and _sustain_ means to continue playing.

<div class="center-stuff">
    <table id="demo_table">
        <thead>
            <tr>
                <th class="table_col">Original Clip</th>
                <th class="table_col">Transformation</th>
                <th class="table_col">Result</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td class="audio_td" data-audio-src="audio/tomic_demos/raw_kick.mp3" desc="Audio - One-Shot Kick Drum"></td>
                <td class="audio_td" style="font-size: small">
                    Type: \(Drum\), Length: 1-bar<br>
                    <pre>bar 1:    ►---►---►---►---</pre>
                    The \(Drum\) transformation  is designed for one-shot drum samples and it replays the clip once for each \(start\) (►) state. In this example, the transformation firstly converts the drum sample to a 1-bar 4/4 kick pattern, then loop the pattern 8 times to match the 8-bar length of the section.
                </td>
                <td class="audio_td" data-audio-src="audio/tomic_demos/processed_kick.mp3" desc="Audio - 4/4 Beat Kick Loop (8-bar)"></td>
            </tr>
            <tr>
                <td class="audio_td" data-audio-src="audio/tomic_demos/raw_clap.mp3" desc="Audio - One-Shot Clap Drum"></td>
                <td class="audio_td" style="font-size: small">
                    Type: \(Drum\), Length: 1-bar<br>
                    <pre>bar 1:    ----►-------►---</pre>
                    This transformation is similar to the previous example but creates a 2/4 beat pattern for the clap sample.
                </td>
                <td class="audio_td" data-audio-src="audio/tomic_demos/processed_clap.mp3" desc="Audio - 2/4 Beat Clap Loop (8-bar)"></td>
            </tr>
            <tr>
                <td class="audio_td" data-audio-src="audio/tomic_demos/raw_muted_guitar.mp3" desc="Audio - Muted Guitar Loop (4-bar)"></td>
                <td class="audio_td" style="font-size: small">
                    Type: \(General\) \(Clip\), Length: 1-bar<br>
                    <pre>bar 1:    ►===============</pre>
                    We want to play the clip in the entire section. This transformation firstly loop itself to match the length of the clip (4-bar), with the first \(start\) state (►) in every segment after the first one being automatically converted to a \(sustain\) state (=). The transformation array then becomes: <br>
                    <pre>►=...= (63 x "=") -> 4 bars total</pre>
                    Next, since the transformation is still shorter than the section length (8-bar), it will loop again to match the section length, but this time the conversion from \(start\) to \(sustain\) is not applied: <br>
                    <pre>►=...= (63 x "=")►=...= (63 x "=") -> 8 bars total</pre>
                    As a result, the 4-bar clip is played 2 times to fill the 8-bar section.
                </td>
                <td class="audio_td" data-audio-src="audio/tomic_demos/processed_muted_guitar.mp3" desc="Audio - Muted Guitar Loop (8-bar)"></td>
            </tr>
            <tr>
                <td class="audio_td" data-audio-src="audio/tomic_demos/raw_lofi_piano.mp3" desc="Audio - LoFi Piano Loop (8-bar)"></td>
                <td class="audio_td" style="font-size: small">
                    Type: \(General\) \(Clip\), Length: 8-bar<br>
                    <pre>bar 1:    ================<br>bar 2:    ================<br>bar 3:    ================<br>bar 4:    ================<br>bar 5:    ►===============<br>bar 6:    ================<br>bar 7:    ================<br>bar 8:    ================<br></pre>
                    We have an 8-bar clip but we want to play only the last 4 bars of it in the section. In this case, we can prepend the \(start\) state with 4 bars of \(sustain\) states. If there is no \(start\) state before a \(sustain\) state, the \(sustain\) also triggers the clip to start playing but muted, the clip will be unmuted until the first \(start\) state.<br>(On the contrary, if there is a \(start\) before the \(sustain\) states, a new \(start\) state will cause the clip to be replayed.)
                </td>
                <td class="audio_td" data-audio-src="audio/tomic_demos/processed_lofi_piano.mp3" desc="Audio - LoFi Piano Loop (8-bar)"></td>
            </tr>
            <tr>
                <td class="audio_td">
                    <div class="midi_wrapper">
                        MIDI - Chord Progression (8-bar)
                        <midi-player src="/audio/tomic_demos/raw_midi.mid" sound-font visualizer="#raw_midi"> </midi-player>
                        <midi-visualizer src="/audio/tomic_demos/raw_midi.mid" type="piano-roll" id="raw_midi"> </midi-visualizer>
                    </div>
                </td>
                <td class="audio_td" style="font-size: small">
                    Type: \(General\) \(Clip\), Length: 4-bar<br>
                    <pre>bar 1:    ►=-=----==-=--=-<br>bar 2:    =-=--=--==-=--=-<br>bar 3:    ==-=----==-=----<br>bar 4:    ==-=--=-==-==-=-</pre>
                    We want to apply some complex chopping to the clip to make it rhythmic. The transformation array is dynamically looped to match the clip's length first (with the first \(start\) state (►) in the second looped segment converted to a \(sustain\) state (=) as well):<br>
                    <pre>bar 1:    ►=-=----==-=--=-<br>bar 2:    =-=--=--==-=--=-<br>bar 3:    ==-=----==-=----<br>bar 4:    ==-=--=-==-==-=-<br>bar 5:    ==-=----==-=--=-<br>bar 6:    =-=--=--==-=--=-<br>bar 7:    ==-=----==-=----<br>bar 8:    ==-=--=-==-==-=-</pre>
                    We use multiple \(rest\) states to mute certain parts to make the clip sounds groovy.
                </td>
                <td class="audio_td">
                    <div class="midi_wrapper">
                        MIDI - Rhythmic Chord Progression (8-bar)
                        <midi-player src="/audio/tomic_demos/processed_midi.mid" sound-font visualizer="#processed_midi"> </midi-player>
                        <midi-visualizer src="/audio/tomic_demos/processed_midi.mid" type="piano-roll" id="processed_midi"> </midi-visualizer>
                    </div>
                </td>
            </tr>
            <tr>
                <td class="audio_td" data-audio-src="audio/tomic_demos/raw_downlifter.mp3" desc="Audio - Downlifter Fx"></td>
                <td class="audio_td" style="font-size: small">
                    Type: \(Fx\)<br>
                    <pre>placement:    START</pre>
                    The \(Fx\) transformation is designed for sound effects like downlifters and uplifters, these samples are placed in either the begining or the end of the section, therefore the \(Fx\) transformation does not require an \(action\) \(sequence\) but a \(placement\) attribute to indicate the clip position. In this case, the downlifter sample is placed in the beginning of the section.
                </td>
                <td class="audio_td" data-audio-src="audio/tomic_demos/processed_downlifter.mp3" desc="Audio - Downlifter Fx (8-bar)"></td>
            </tr>
            <tr>
                <td class="audio_td" data-audio-src="audio/tomic_demos/raw_sweep.mp3" desc="Audio - SweepUp Fx"></td>
                <td class="audio_td" style="font-size: small">
                    Type: \(Fx\)<br>
                    <pre>placement:    END</pre>
                    The SweepUp clip needs to be placed at the end of the section, which means the clip end should be aligned with the section end, we can simply use this transformation to dynamically calculate the correct start timing for each associated clip.
                </td>
                <td class="audio_td" data-audio-src="audio/tomic_demos/processed_sweep.mp3" desc="Audio - SweepUp Fx (8-bar)"></td>
            </tr>
        </tbody>
    </table>
    After assigning tracks to the above transformed clips, we can achieve the same music representation on a digital audio workstation:
    <div class="center-stuff"><img src="/assets/pics/daw_representation.jpg" style="width:60%" alt=""></div>
    Now, let's take a listen to the full section (8-bar):
    <div class="audio_wrapper">
        <button id="play_btn_full_demo" class="play_btn">▶</button>
        <div id="waveform_full_demo" class="waveform"></div>
    </div>
    <script>
        const wavesurfer = WaveSurfer.create({
            container: `#waveform_full_demo`,
            waveColor: '#B1B1B1',
            progressColor: '#F6B094',
            barWidth: 2,
            interact: true,
            pixelRatio: 1,
            height: 40,
            cursorWidth: 2,
            cursorColor: "red",
            url: "/audio/tomic_demos/full_section.mp3",
            plugins: [
                WaveSurfer.Hover.create({
                    lineColor: '#ff0000',
                    lineWidth: 2,
                    labelBackground: '#555',
                    labelColor: '#fff',
                    labelSize: '11px',
                }),
            ],
        });
        let btnEle = document.getElementById("play_btn_full_demo")
        // 绑定播放按钮事件
        btnEle.addEventListener("click", function () {
            if (btnEle.textContent === "▶") {
                stopPlaying();
                btnEle.textContent = '◼';
                wavesurfer.play();
                currentPlayingAudio = wavesurfer;
                currentPlayingBtn = btnEle;
            } else {
                stopPlaying();
            }
        });
    </script>
</div>
<br>

---

## Music Demos
We compare $$e\text{TOMIC}$$ with MusicGen and two ablations in the generation of electronic music. For exporting audio using REAPER, there are 8 virtual instrument presets (5 for chord, 2 for melody, and 1 for bass) for MIDI tracks to choose from randomly. We keep all REAPER settings to default, and no mixing plug-ins are applied except for a limiter on the master track to prevent audio clipping.

**MusicGen:**
We use MusicGen-Large-3.3B model as the baseline, with prompts that specify tonality, tempo, and song structure. It helps to evaluate our approach against state-of-the-art music generation systems. Given MusicGen's 30-second generation limit, we implement a sliding window approach to generate longer audio by using a fixed 30-second window that slides in 10-second chunks, while using the previous generated 20 seconds as context. To enable structural awareness during generation, we modify the model's inference process by appending explicit structure context after the initial text prompt at each generation step, instructing the model to align its output with the given structure. A prompt example is as follows:

>Full prompt of one generation step:\
>(1) Initial Prompt: Generate an electronic track at 120 BPM in C major. Follow this structure: 8-bar intro, 16-bar verse, 8-bar pre-chorus, 16-bar chorus…\
>(2) Structure Context: For now, you are generating the segment between the 120th second and the 150th second, which corresponds to the the 60th bar and the 75th bar regarding the whole song, your generated segment includes 4 bars of PreChorus, 8 bars of Chorus, and 3 bars of Outro of the song structure.

**Standalone LLM ($$e\textbf{TOMIC}$$ w/o Composition Links):**
This ablation uses the same GPT-4o model but without our composition links architecture. It helps to evaluate the impacts of our composition links representation on generation quality. We design the prompt to let the LLM generate a sequence of tracks and clip descriptions with position information (time point and track location) conditioned on a song structure, and then the content retrieval mechanism is also applied for the clips; lastly, we convert the generated output to a REAPER arrangement for audio rendering.

**Random ($$e\textbf{TOMIC}$$ w/o LLM):**
To assess the contribution of LLM-based generation, we implement a rule-based ablation that uses the same $$e\text{TOMIC}$$ structure but generates arrangements through a series of randomized operations. This method first creates a random number (15-25) of track nodes, then populates sections with clips based on stochastic decisions. For each track on each unique section, the system determines clip placement through a two-step random process: first deciding whether to place a clip, if so, then choosing between reusing an existing clip or generating a new one. For each MIDI clip, the content type is randomly assigned one of three types (chord, bass, or melody), with bass clips comprise root notes derived from existing chord clips to maintain coherence. The generation of audio clips involves random selection from a predefined set of feature tags, which cover tonal elements, percussion, and sound effects. The final step links one of four predefined transformation nodes to each clip based on the clip's content type or audio features.

#### **Generation Results**
We predefine 4 tonalities (C major / A minor, F major / D minor, G major / E minor and B$$\flat$$ major / G minor) and four distinct song structures, each comprising a sequence of section names with phrase labels and bar durations. Sections with the same name and phrase label should be identical (eg. $$\text{pre-chorus}$$ and $$\text{pre-chorus}$$), while those with different names but the same phrase label should be similar (eg. $$\text{verse 1}$$ and $$\text{verse 2}$$). Then, we generate four sets of electronic music compositions of 120 BPM for each method, with each set containing 8 compositions generated under the same song structure and 4 different tonalities (2 compositions for each key).

##### <center> \( \textbf{Structure 1: } \) \( \text{intro(8b)} \) \( \rightarrow \) \( \text{verse 1(16b)} \) \( \rightarrow \) \( \text{pre-chorus(8b)} \) \( \rightarrow \) \( \text{chorus 1(16b)} \) \( \rightarrow \) \( \text{verse 2(16b)} \) \( \rightarrow \) \( \text{pre-chorus(8b)} \) \( \rightarrow \) \( \text{chorus 2(16b)} \) \( \rightarrow \) \( \text{bridge(8b)} \) \( \rightarrow \) \( \text{chorus 3(16b)} \) \( \rightarrow \) \( \text{outro(8b)} \)
<div class="center-stuff">
<table id="table_pattern1">
    <thead>
        <tr>
            <th class="slash-wrap">
                <span class="left">Key</span>
                <span class="slash"></span>
                <span class="right">Structure</span>
            </th>
            <th class="table_col">MusicGen</th>
            <th class="table_col">Standalone LLM</th>
            <th class="table_col">Random</th>
            <th class="table_col">\( e \)TOMIC</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td class="col-name" rowspan="2">C major<br>/<br>A minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern1/pattern1_C_1.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern1/pattern1_C_1.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern1/pattern1_C_1.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern1/pattern1_C_1.mp3" data-pattern="pattern1"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern1/pattern1_C_2.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern1/pattern1_C_2.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern1/pattern1_C_2.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern1/pattern1_C_2.mp3" data-pattern="pattern1"></td>
        </tr>
        <tr>
            <td class="col-name" rowspan="2">F major<br>/<br>D minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern1/pattern1_F_1.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern1/pattern1_F_1.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern1/pattern1_F_1.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern1/pattern1_F_1.mp3" data-pattern="pattern1"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern1/pattern1_F_2.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern1/pattern1_F_2.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern1/pattern1_F_2.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern1/pattern1_F_2.mp3" data-pattern="pattern1"></td>
        </tr>
        <tr>
            <td class="col-name" rowspan="2">G major<br>/<br>E minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern1/pattern1_G_1.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern1/pattern1_G_1.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern1/pattern1_G_1.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern1/pattern1_G_1.mp3" data-pattern="pattern1"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern1/pattern1_G_2.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern1/pattern1_G_2.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern1/pattern1_G_2.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern1/pattern1_G_2.mp3" data-pattern="pattern1"></td>
        </tr>
        <tr>
            <td class="col-name" rowspan="2">B\( \flat \) major<br>/<br>G minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern1/pattern1_A%23_1.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern1/pattern1_A%23_1.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern1/pattern1_A%23_1.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern1/pattern1_A%23_1.mp3" data-pattern="pattern1"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern1/pattern1_A%23_2.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern1/pattern1_A%23_2.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern1/pattern1_A%23_2.mp3" data-pattern="pattern1"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern1/pattern1_A%23_2.mp3" data-pattern="pattern1"></td>
        </tr>
    </tbody>
</table>
</div>

##### <center> \( \textbf{Structure 2: } \) \( \text{intro(8b)} \) \( \rightarrow \) \( \text{verse 1(16b)} \) \( \rightarrow \) \( \text{chorus 1(8b)} \) \( \rightarrow \) \( \text{verse 2(16b)} \) \( \rightarrow \) \( \text{chorus 2(8b)} \) \( \rightarrow \) \( \text{bridge(8b)} \) \( \rightarrow \) \( \text{chorus 3(8b)} \) \( \rightarrow \) \( \text{outro(8b)} \)
<div class="center-stuff">
<table id="table_pattern2">
    <thead>
        <tr>
            <th class="slash-wrap">
                <span class="left">Key</span>
                <span class="slash"></span>
                <span class="right">Structure</span>
            </th>
            <th class="table_col">MusicGen</th>
            <th class="table_col">Standalone LLM</th>
            <th class="table_col">Random</th>
            <th class="table_col">\( e \)TOMIC</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td class="col-name" rowspan="2">C major<br>/<br>A minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern2/pattern2_C_1.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern2/pattern2_C_1.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern2/pattern2_C_1.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern2/pattern2_C_1.mp3" data-pattern="pattern2"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern2/pattern2_C_2.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern2/pattern2_C_2.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern2/pattern2_C_2.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern2/pattern2_C_2.mp3" data-pattern="pattern2"></td>
        </tr>
        <tr>
            <td class="col-name" rowspan="2">F major<br>/<br>D minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern2/pattern2_F_1.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern2/pattern2_F_1.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern2/pattern2_F_1.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern2/pattern2_F_1.mp3" data-pattern="pattern2"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern2/pattern2_F_2.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern2/pattern2_F_2.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern2/pattern2_F_2.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern2/pattern2_F_2.mp3" data-pattern="pattern2"></td>
        </tr>
        <tr>
            <td class="col-name" rowspan="2">G major<br>/<br>E minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern2/pattern2_G_1.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern2/pattern2_G_1.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern2/pattern2_G_1.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern2/pattern2_G_1.mp3" data-pattern="pattern2"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern2/pattern2_G_2.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern2/pattern2_G_2.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern2/pattern2_G_2.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern2/pattern2_G_2.mp3" data-pattern="pattern2"></td>
        </tr>
        <tr>
            <td class="col-name" rowspan="2">B\( \flat \) major<br>/<br>G minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern2/pattern2_A%23_1.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern2/pattern2_A%23_1.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern2/pattern2_A%23_1.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern2/pattern2_A%23_1.mp3" data-pattern="pattern2"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern2/pattern2_A%23_2.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern2/pattern2_A%23_2.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern2/pattern2_A%23_2.mp3" data-pattern="pattern2"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern2/pattern2_A%23_2.mp3" data-pattern="pattern2"></td>
        </tr>
    </tbody>
</table>
</div>

##### <center> \( \textbf{Structure 3: } \) \( \text{intro(8b)} \) \( \rightarrow \) \( \text{verse 1(16b)} \) \( \rightarrow \) \( \text{pre-chorus 1(8b)} \) \( \rightarrow \) \( \text{chorus 1(16b)} \) \( \rightarrow \) \( \text{verse 2(16b)} \) \( \rightarrow \) \( \text{pre-chorus 2(8b)} \) \( \rightarrow \) \( \text{chorus 2(16b)} \) \( \rightarrow \) \( \text{bridge(8b)} \) \( \rightarrow \) \( \text{chorus 3(16b)} \) \( \rightarrow \) \( \text{outro(8b)} \)
<div class="center-stuff">
<table id="table_pattern3">
    <thead>
        <tr>
            <th class="slash-wrap">
                <span class="left">Key</span>
                <span class="slash"></span>
                <span class="right">Structure</span>
            </th>
            <th class="table_col">MusicGen</th>
            <th class="table_col">Standalone LLM</th>
            <th class="table_col">Random</th>
            <th class="table_col">\( e \)TOMIC</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td class="col-name" rowspan="2">C major<br>/<br>A minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern3/pattern3_C_1.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern3/pattern3_C_1.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern3/pattern3_C_1.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern3/pattern3_C_1.mp3" data-pattern="pattern3"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern3/pattern3_C_2.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern3/pattern3_C_2.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern3/pattern3_C_2.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern3/pattern3_C_2.mp3" data-pattern="pattern3"></td>
        </tr>
        <tr>
            <td class="col-name" rowspan="2">F major<br>/<br>D minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern3/pattern3_F_1.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern3/pattern3_F_1.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern3/pattern3_F_1.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern3/pattern3_F_1.mp3" data-pattern="pattern3"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern3/pattern3_F_2.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern3/pattern3_F_2.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern3/pattern3_F_2.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern3/pattern3_F_2.mp3" data-pattern="pattern3"></td>
        </tr>
        <tr>
            <td class="col-name" rowspan="2">G major<br>/<br>E minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern3/pattern3_G_1.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern3/pattern3_G_1.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern3/pattern3_G_1.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern3/pattern3_G_1.mp3" data-pattern="pattern3"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern3/pattern3_G_2.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern3/pattern3_G_2.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern3/pattern3_G_2.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern3/pattern3_G_2.mp3" data-pattern="pattern3"></td>
        </tr>
        <tr>
            <td class="col-name" rowspan="2">B\( \flat \) major<br>/<br>G minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern3/pattern3_A%23_1.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern3/pattern3_A%23_1.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern3/pattern3_A%23_1.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern3/pattern3_A%23_1.mp3" data-pattern="pattern3"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern3/pattern3_A%23_2.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern3/pattern3_A%23_2.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern3/pattern3_A%23_2.mp3" data-pattern="pattern3"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern3/pattern3_A%23_2.mp3" data-pattern="pattern3"></td>
        </tr>
    </tbody>
</table>
</div>

##### <center> \( \textbf{Structure 4: } \) \( \text{intro(8b)} \) \( \rightarrow \) \( \text{chorus(8b)} \) \( \rightarrow \) \( \text{verse(16b)} \) \( \rightarrow \) \( \text{pre-chorus(4b)} \) \( \rightarrow \) \( \text{chorus(8b)} \) \( \rightarrow \) \( \text{verse(16b)} \) \( \rightarrow \) \( \text{pre-chorus(4b)} \) \( \rightarrow \) \( \text{chorus(8b)} \) \( \rightarrow \) \( \text{outro(8b)} \)
<div class="center-stuff">
<table id="table_pattern4">
    <thead>
        <tr>
            <th class="slash-wrap">
                <span class="left">Key</span>
                <span class="slash"></span>
                <span class="right">Structure</span>
            </th>
            <th class="table_col">MusicGen</th>
            <th class="table_col">Standalone LLM</th>
            <th class="table_col">Random</th>
            <th class="table_col">\( e \)TOMIC</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td class="col-name" rowspan="2">C major<br>/<br>A minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern4/pattern4_C_1.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern4/pattern4_C_1.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern4/pattern4_C_1.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern4/pattern4_C_1.mp3" data-pattern="pattern4"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern4/pattern4_C_2.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern4/pattern4_C_2.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern4/pattern4_C_2.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern4/pattern4_C_2.mp3" data-pattern="pattern4"></td>
        </tr>
        <tr>
            <td class="col-name" rowspan="2">F major<br>/<br>D minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern4/pattern4_F_1.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern4/pattern4_F_1.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern4/pattern4_F_1.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern4/pattern4_F_1.mp3" data-pattern="pattern4"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern4/pattern4_F_2.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern4/pattern4_F_2.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern4/pattern4_F_2.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern4/pattern4_F_2.mp3" data-pattern="pattern4"></td>
        </tr>
        <tr>
            <td class="col-name" rowspan="2">G major<br>/<br>E minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern4/pattern4_G_1.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern4/pattern4_G_1.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern4/pattern4_G_1.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern4/pattern4_G_1.mp3" data-pattern="pattern4"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern4/pattern4_G_2.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern4/pattern4_G_2.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern4/pattern4_G_2.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern4/pattern4_G_2.mp3" data-pattern="pattern4"></td>
        </tr>
        <tr>
            <td class="col-name" rowspan="2">B\( \flat \) major<br>/<br>G minor</td>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern4/pattern4_A%23_1.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern4/pattern4_A%23_1.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern4/pattern4_A%23_1.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern4/pattern4_A%23_1.mp3" data-pattern="pattern4"></td>
        </tr>
        <tr>
            <td class="audio_td" data-audio-src="audio/musicgen/pattern4/pattern4_A%23_2.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/llm/pattern4/pattern4_A%23_2.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/random/pattern4/pattern4_A%23_2.mp3" data-pattern="pattern4"></td>
            <td class="audio_td" data-audio-src="audio/etomic/pattern4/pattern4_A%23_2.mp3" data-pattern="pattern4"></td>
        </tr>
    </tbody>
</table>
</div>

---
## REAPER Digital Audio Workstation Integration
In this video, we initiate the generation process and show the results directly in REAPER. Then, we manually adjusted 
some virtual instruments and mixing parameters to show the user co-creation capability.
<div class="center-stuff">
    <video controls width="800" src="demo.mp4"></video>
</div>

<br>

Thanks <a href="https://cifkao.github.io/html-midi-player/">html-midi-player</a> for the excellent MIDI visualization.