+++
title = "Talking with robots"
date = 2022-04-08

[taxonomies]
tags = ["rust", "stm32", "embedded", "DSP"]
categories = ["Tech"]
+++

I always found configuring wireless gadgets very annoying.

Pairing/unpairing/repairing bluetooth things on badly designed user interfaces is not my thing.

But what are the options for simple wireless communication?

<!-- more -->

A surprisingly non-obvious option is the use of sound waves to create a sort of communication. Surprisingly because that is what we use for talking with each other, but not something we consider to use to interact with an embedded electronic system.

Developing speech recognition is not a simple task. But maybe this is not the correct path to achieve a communication channel with hardware for simple applications. Detecting single tones is a very feasible thing to do though.

A handy algorithm that can be used to detect tones even on memory constrained hardware is the [Goertzel Algorithm](https://en.wikipedia.org/wiki/Goertzel_algorithm). Opposite to Fast Fourier Transform, It requires much less memory and CPU cycles to compute a predefined set of frequencies.

Based on those assumptions, decided to prototype a robot controlled by sound wave tones.

This is what I achieved:

<iframe width="560" height="315" src="https://www.youtube.com/embed/JwpZoJkUezs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

The embedded Rust source running on the STM32 bluepill microcontroller is available at [https://github.com/arturaugusto/audio-drone](https://github.com/arturaugusto/audio-drone).

For my robot, I defined four frequencies to be detected. Each frequency is associated with some action, like move forward, backward, turn left or right.

I decided to write my own implementation of Goertzel Algorithm in Rust. It is almost a verbatim translation from [this C implementation](https://netwerkt.wordpress.com/2011/08/25/goertzel-filter/). The source code is available at [https://github.com/arturaugusto/rt-goertzel](https://github.com/arturaugusto/rt-goertzel)

The follow snippet is a self-contained full usage example of implemented algorithm that can be run on [https://play.rust-lang.org/](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=9fa02a5c291bdba3687129bcfd7733af):

```rust
pub mod filter {
    pub struct Goertzel {
        s_prev: f32,
        s_prev2: f32,
        totalpower: f32,
        n: i32,
        mean: f32,
        mean_prev: f32,

        freq: f32,
        samplef: f32
    }

    impl Goertzel {
        pub fn new(freq: f32, samplef: f32) -> Self {
            Self {
                s_prev: 0.,
                s_prev2: 0.,
                totalpower: 0.,
                n: 0,
                mean: 0.,
                mean_prev: 0.,

                freq: freq,
                samplef: samplef
            }
        }
        
        pub fn filter (&mut self, sample: f32) -> f32 {
            // real time mean
            // https://dsp.stackexchange.com/questions/811/determining-the-mean-and-standard-deviation-in-real-time
            self.mean_prev = self.mean;
            self.n = self.n + 1;
            self.mean = self.mean + (sample-self.mean)/self.n as f32;
            let x: f32 = sample - self.mean;

            let normalizedfreq: f32 = self.freq/self.samplef;
            let coeff: f32 = 2.*(2.*3.1416*normalizedfreq).cos();
            let s: f32 = x + coeff * self.s_prev - self.s_prev2;
            self.s_prev2 = self.s_prev;
            self.s_prev = s;

            let power: f32 = self.s_prev2*self.s_prev2+self.s_prev*self.s_prev-coeff*self.s_prev*self.s_prev2;
            self.totalpower = self.totalpower + x*x;
            if self.totalpower < 0. {
                self.totalpower = 1.
            }
            return power / self.totalpower / self.n as f32
        }
    }
}

use crate::filter::Goertzel;
use core::f32::consts::PI;

fn main() {

  let target_freq = 100.;
  let sample_freq = 1000.;
  let mut g1 = Goertzel::new(target_freq, sample_freq);
  let mut g2 = Goertzel::new(target_freq, sample_freq);
  
  
  let mut res_100 = 0.;
  let mut res_99 = 0.;
  for n in 0..1000 {
    let sample =  (n as f32 * 1./sample_freq * 2. * PI * target_freq).sin();
    res_100 = g1.filter(sample);
    let sample =  (n as f32 * 1./sample_freq * 2. * PI * 99.).sin();
    res_99 = g2.filter(sample);
  }
  println!("Non zero value means that filter detected some energy.");
  println!("Result for exact filter frequency: {:?}", res_100);
  println!("Result for a close but not exact filter frequency: {:?}", res_99);
}
```

The output of this snippet is:

```
Non zero means that some energy was detected.
Result for exact filter frequency: 0.49815935
Result for a close but not exact filter frequency: 1.9859854e-5
```

I also made a simple [user interface](https://github.com/arturaugusto/audio-drone-keypad) using Javascript and the browser audio API to play tones, so I don't need to whistle every time like here:

<iframe width="560" height="315" src="https://www.youtube.com/embed/rYuAQTtDHGI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


