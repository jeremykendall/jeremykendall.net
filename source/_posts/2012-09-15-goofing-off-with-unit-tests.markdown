---
author: admin
comments: true
date: 2012-09-15 02:59:08+00:00
layout: post
slug: goofing-off-with-unit-tests
title: Goofing off with unit tests
wordpress_id: 427
categories:
- Development
- Personal
tags:
- humor
- Melissa
- Mercyful Fate
- music
- phpunit
- unit test
---

[_**UPDATE**: The library and test have been refactored. See the [commit](https://github.com/jeremykendall/mercyful-fate/commit/2325faa0192bc777ac5fbbb92cdc871d459cfd60) for full details._]

Sometime last year I saw a unit test for the song "[Ice Cream Paint Job](http://www.youtube.com/watch?v=0yfArN-e2OU)".  I thought it was hilarious, and I hate I've never been able to find it again.  I liked it so much, in fact, I decided to write my own musical <del>unit</del> integration test.  Behold the MelissaTest, a short, simple test covering the main premise of "[Melissa](http://www.youtube.com/watch?v=OVVvVFmoHnE)".  Bonus points for me: [_the test passes_](https://github.com/jeremykendall/mercyful-fate).  Enjoy.


    
    
    king = new KingDiamond();
            $this->priest = new Priest();
            $this->priest->attach($this->king);
            $this->witch = new WitchMelissa();
            $this->trackMelissa = new TrackMelissa($this->king, $this->witch, $this->priest);
        }
        
        protected function tearDown()
        {
            $this->trackMelissa = null;
        }
    
        public function testBurnMelissa()
        {
            $this->assertFalse($this->witch->isBurned());
            $this->assertFalse($this->king->swearsRevenge());
    
            $this->trackMelissa->priestBurnsWitch();
            
            $this->assertTrue($this->witch->isBurned());
            $this->assertTrue($this->king->swearsRevenge());
        }
    
    }
    
