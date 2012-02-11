image-deduplication-tool: simple tool to detect (and get rid of) similar images using perceptual hashing
--------------------

There's gonna be a lot of duplicates in almost any arbitrary collection of
images, it can be actually surprising how many.

pHash lib, which is the core of the tool easily detects cropped and retouched
images, same thing in different resolutions and formats.

Tool itself goes over the specified paths, calculating the hashes of all the
images there, pickling them into a file between runs (to save lots of time on
re-calculating all of them).
Then it just compares the hashes, showing closest results first.

pHash lib seem to be able to utilize multiple cpu cores for the hashing when
built with with openmp flag, but I wasn't able to use that, so put much simplier
and robust solution in place to scale such task - just forking worker pid for
each hardware thread.

Optinally, the thing starts handy [feh](http://derf.homelinux.org/projects/feh/)
viewer, where human can make a decision to remove one image version or the other
(with pre-configured "rm" action), skip to the next pair or stop the comparison.


Requirements
--------------------

* [python 2.7](http://python.org) (with ctypes support)
* [libpHash](http://phash.org) (used via ctypes)
* (optional) [feh image viewer](http://derf.homelinux.org/projects/feh/)


Usage
--------------------

Just run as `./image_matcher.py --feh ~/media/images`.

% ./image_matcher.py -h

    usage: image_matcher.py [-h] [--hash-db PATH] [-p THREADS] [-n COUNT] [--feh]
                            [--feh-args CMDLINE] [--debug]
                            paths [paths ...]

    positional arguments:
      paths                 Paths to match images in (can be files or dirs).

    optional arguments:
      -h, --help            show this help message and exit
      --hash-db PATH        Path to db to store hashes in (default:
                            image_matcher.db).
      -p THREADS, --parallel THREADS
                            How many hashing ops can be done in parallel (default:
                            try cpu_count() or 1).
      -n COUNT, --top-n COUNT
                            Limit output to N most similar results (default: print
                            all).
      --feh                 Run feh for each image match with removal actions
                            defined (see --feh-args).
      --feh-args CMDLINE    Feh commandline parameters (space-separated, unless
                            quoted with ") before two image paths (default: -GNFY
                            --info "echo '%f %wx%h (diff: {diff})'" --action8 "rm
                            %f" --action1 "kill -INT {pid}", only used with --feh,
                            python-format keywords available: path1, path2, pid,
                            diff)
      --debug               Verbose operation mode.

feh can be customized to do any custom action or show any kind of info alongside
images with --feh-args parameter. It's also possible to make it show images
side-by-side in montage mode or in separate windows in multiwindow mode, see
"man feh" for details.

Default line (`feh -GNFY --info "echo '%f %wx%h (diff: {diff})'" --action8 "rm
%f" --action1 "kill -INT {pid}" {path1} {path2}`) makes it show fullscreen
image, some basic info (along with difference between image hashes) about it and
action reference, pressing "8" there will remove currently displayed version,
"1" will stop the comparison and quitting feh ("q") will go to the next pair.


Known Issues
--------------------

pHash seem to be prone to hanging indefinitely on some non-image files without
consuming much resources. Use `./image_matcher.py --debug -p 1` to see on which
exact file it hangs in such cases.

I'll probably add some check for file magic to see if it's image before running
pHash over it in the future.

pHash also gives zero as a hash value for some images. No idea why it does that
atm, but these "0" values obviously can't be meaningfully compared to anything,
so tool skips them, issuing a log message (seen only with --debug).