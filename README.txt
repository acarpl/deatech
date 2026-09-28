YOUR DEMO — 3 easy steps

What's in this folder:
  index.html   <- your demo (open this to edit)
  image.jpg    <- example image (you can replace it)
  dummy-bw/    <- the AI "brain" (don't delete)

-----------------------------------------------------
STEP 1 — RUN IT (through localhost, NOT double-click)
-----------------------------------------------------
The camera WON'T work if you just double-click the file. It has to run through "localhost".
Pick one:

  Option A (easiest) — VS Code:
    1. Install the "Live Server" extension in VS Code
    2. Right-click index.html  ->  "Open with Live Server"
    3. The browser opens by itself. Done.

  Option B — Terminal:
    1. Open a Terminal in this folder
    2. Type:   python -m http.server      (or: python3 -m http.server)
    3. Open your browser at:   http://localhost:8000

-----------------------------------------------------
STEP 2 — TEST
-----------------------------------------------------
Click "Start Camera" -> allow the camera -> cover the camera / show a dark object.
The trigger fires. It works!

-----------------------------------------------------
STEP 3 — MAKE IT YOURS (open index.html in a text editor)
-----------------------------------------------------
Find the "CHANGE THIS" part and edit:
  TEXT  = your text
  IMAGE = your image file name (put it in this folder)
  SOUND = true (with sound) / false (silent)
Save, refresh the browser. Now it's your own.

-----------------------------------------------------
LEVEL 2 (optional) — YOUR OWN AI MODEL
-----------------------------------------------------
1. Go to teachablemachine.withgoogle.com -> New Project -> Image -> Standard
2. Make a few classes, record samples with your webcam, click Train Model
3. Export Model -> Tensorflow.js -> Upload (shareable link) -> copy the link
4. In index.html under "LEVEL 2", change:
     MODEL         = your link (with a "/" at the end)
     TRIGGER_CLASS = the class name that should trigger (EXACTLY as in TM)

Stuck? Open: morari.studio/kelas-c/session-3/build/
