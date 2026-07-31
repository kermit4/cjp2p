# Make Your Own P2P App With AI and No Programming Experience

*No coding experience required, no computer required. Just your phone.*

By the end of this, you'll have invented a small app -- a game, a chat room, a
dice roller, whatever you want -- and it will be running on your phone,
talking directly to other people's phones. No company in the middle, no
account, no sign-up, nobody's server. An AI (Meta AI) will write the actual
code for you. Your job is just to describe what you want and tap the buttons
in this guide.

A word you'll see a lot, defined once up front:

- *Peer-to-peer (P2P)* -- devices talking directly to each other over the
  internet, instead of everyone going through one company's server. If that
  company's server goes down, gets bought, or decides to ban you, a P2P app
  keeps working, because there was never a middleman to begin with.
- *Node* -- the small app you're about to install. It runs quietly in the
  background and does the work of finding and talking to other people's
  phones running the same app.

Everything below happens on your phone (Android for now -- iPhone support
isn't there yet). No terminal, no typing commands.

---

## Step 1 -- Get Meta AI, the AI that will write your code

Meta AI is an app/website you type requests into ("write me a game where...")
and it writes real, working computer code back. You don't need to understand
the code it writes any more than you need to understand an engine to drive a
car.

Go here on your phone or computer: *https://www.meta.ai/*

You can use it on the web with no install, or install the Meta AI app. Sign in and keep it handy. You'll come back to it in Step 3.

---

## Step 2 -- Install your node

1. On your Android phone, open this link and download the file:
   *https://github.com/kermit4/cjp2p-rust/releases/latest/download/app-universal-release.apk*
2. Open the downloaded file. Android will probably warn you about installing
   from outside the Play Store ("install unknown apps") -- that's just
   Android being cautious about anything that isn't from its own store, not a
   warning specific to this app. Allow it and tap install.
3. Open the app. It quietly starts your node in the background and shows you
   its home page -- a list of small P2P apps other people have already made
   with this exact tool: a chat room, a real-time 2-player game, a video
   call, a collaborative digital audio workstation, and more.

Nothing you're about to do gets uploaded to a company. This page is your
phone talking to itself first, and other people's phones or computers second.

Click around a few of the examples if you like. Every one of them is a
single, plain HTML file -- the same kind of file a web browser reads to show
you any website -- and every one was written by asking an AI to make it,
exactly like you're about to do. 

---

## Step 3 -- Ask Meta AI to build you something

Open Meta AI at *https://www.meta.ai/*. Copy the block below and change the description in the middle
to whatever you want -- a game, a shared to-do list, a drawing board, a poll
for your friends, anything.
I'm running a peer-to-peer node app called cjp2p on my phone (from
https://github.com/kermit4/cjp2p-rust). It lets web pages talk to each
other directly over the internet through a WebSocket at ws://localhost:24255/wt,
with no server and no account.

Look at the example apps here for how the networking works:
https://github.com/kermit4/LCDP_web_apps

Using that same approach, build me a single, self-contained HTML file
(no external files, no build step, no server code) for:

    <<< describe what you want here -- e.g. "a shared drawing board where
    everything I draw shows up on my friend's screen instantly" or
    "a two-player tic-tac-toe game" or "a dice roller both of us can see" >>>

Output just the one HTML file.
Meta AI will write back one long file. You don't need to read it or
understand it. Tap the download / save button, or copy the text into a notes app and
save it as a file with a name ending in `.html` -- for example
`MyDiceRoller.html` -- somewhere you can find it, like your phone's
Downloads.

If Meta AI shows you a preview instead of code, say: "give me the file as a downloadable .html file"

---

## Step 4 -- Publish it, no terminal needed

Back in your node app, find and open *publish.html* from the list on the
home page. This page lets you hand it a file with your finger, no typing
required:

1. Tap *pick files* and choose the `.html` file you just saved.
2. Tap *Publish*.
3. You'll get a link back that looks like
   `http://127.0.0.1:24255/latest/<your-key>/MyDiceRoller.html`. Tap it --
   that's your app, running.

That's it -- you made a P2P app, and you never touched a terminal or wrote a
line of code.

### Try it with a friend

Have your friend do Step 2 too, so they get their own node running on their
own phone (everyone runs their own copy). If the app you asked Meta AI for
shows a "share this link" box on screen once it's running, send your friend
that link. They open it on their phone, and their node fetches your published
page for them automatically -- still no server, no sign-up, nothing in the
middle.

---

## If something doesn't work

- *Android won't install the app / blocks it* -- look for "install unknown
  apps" or "allow from this source" in the install prompt or in
  Settings -- Apps, and allow it for whichever app you downloaded the file
  with (browser or file manager).
- *The home page never loads after opening the app* -- give it a few
  seconds; it's starting the node in the background. If it still doesn't
  load, close the app fully and reopen it.
- *Publish button doesn't seem to do anything* -- make sure you tapped
  "pick files" and actually chose your `.html` file first; the Publish
  button only appears once a file is staged.
- *Your app doesn't show your friend, or theirs doesn't show you* -- give
  it 10-20 seconds; nodes take a moment to find each other over the internet.
  If it still doesn't work, copy the exact error text (from the page, if you
  see one) and paste it back to Meta AI along with "this isn't working, here's
  the error" -- that's the normal way to fix it, and it works even when
  neither of you understands the error.

You never need to get this perfect on the first try. Pasting an error back to
Meta AI and asking it to fix it is exactly what people who've coded for
decades do too.
