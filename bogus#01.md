# Bogus
## The basics

**Bogus** is a library that aims to reduce the risks associated with **isolated unit testing**.
Here is an example of using plain old stubs. Nothing too fancy...

    describe Mission do
      it "checks protocol for 007’s mission start" do
        briefing = stub( meeting_with_M: [top_secret_briefing]) 
        gadgets  = stub( meeting_with_Q: [stylish_Aston_Martin, laser_Rolex])
        mission_start = Mission.new(briefing, gadgets)
        mission_start.check_protocol.should == mission_is_a_go
      end
    end

## The Problem
It can be summarized by “*Programming by wishful thinking*.”
Classes under test not only know how methods work, they also specify how tested classes will interact with it’s collaborators and what those collaborators will be. This *top-down system design*, where classes get implemented before other collaborators exist yet, seems to be a very good thing in principle. You only implement what you need.

## BUT!
The same freedom from not having to implement collaborators can quickly turn on you.
Once the tested class is implemented – *what tells you that the collaborators aren’t implemented yet? What tells your previous tests that the interface for their collaborators have changed*?
### In Plain English
This way of stubbing requires you to write integration or end-to-end tests because you wanna make sure the pieces fit together. The problem with those integration tests is that they unnecessarily add slower and harder to set up tests to cover all the integration points between the objects/collaborators.

Another potential problem lurks in the dark because those stubbed collaborators will likely be **interacting with more objects than the tested class**. This obviously opens the door wide open for **duplication** and we all know what awaits us if we wanna change the collaborators later on... Remember all the places where you created that stub? A headache Bogus helps to prevent. Worst of all, your tests won’t help – *they don’t have a clue what the collaborators interface should look like*.

## THE SOLUTION
Bogus makes your test doubles more ASSERTIVE!
They will tell you:
“*Hey, you are stubbing the wrong thing here*!”
“*No, that’s not the way how the interface looks like*!”

* It does not only make sure the stubbed class work in your tests.
* It also assures those stubbed collaborators classes EXIST and have an INTERFACE.
* Finally it checks the the number of arguments provided.

The syntax is quite simple and similar to regular stubs:

    stub(meeting_with_M: [top_secret_briefing])
    fake(:M, meeting_with_M: [top_secret_briefing])
		
=> faked **class M exits** now and has an **interface** through #meeting_with_M and **returns** [top_secret_briefing]

To test the **Mission class** you need **3** classes – 2 of them stubbed out:
**Mission, M and Q**.

    describe Mission do
      it "checks protocol for 007’s mission start" do
        briefing = fake(:M, meeting_with_M: [top_secret_briefing])
        gadgets  = fake(:Q, meeting_with_Q: [stylish_Aston_Martin, laser_Rolex])

        mission_start = Mission.new(briefing, gadgets)
        mission_start.check_protocol.should == mission_is_a_go
	    end
    end

Very similar syntax that you’re used to but adds a lot of goodies.
This creates the M and Q classes and provide an interface. 
Of course the question arises – “How much **DRYer** is this solution really?”


The neat thing you can do to DRY your stubs is **pulling them out in a fakes.rb file** (spec/support/fakes.rb) You can put methods that return values in it. All you need to do is provide a **reasonable default return value** for those methods.

The syntax is straight forward and looks like this:

    Bogus.fakes do
      fake(:m) do
        drop_a_random_lexical_question_on_007 ["Well done James!"]
      end

      fake(:james_bond) do
        asked_random_lexical_question [witty_answer]
      end
    end
	
This creates fakes for the **M** and **JamesBond** classes and provide methods with return values. If you want to change the fakes you can do it now in **one central place** and the fakes in your tests get updated. Pretty cool! 

In your tests you only have to tell your test blocks what fake you wanna use:

    describe MissionBriefing do
      let(:tricky_question) { "How many agents were killed in the James Bond series?" }

      describe M do
        fake(:james_bond)
				verify_contract(:james_bond)

        it "drops random lexical questions on 007 "
          JamesBond.asked_lexical_question(tricky_question)			
          M.should have_received(witty_answer)
        end

      describe JamesBond do
        fake(:m)

        it "impress M with random knowledge bombs" do
          M.drop_a_random_lexical_question_on_007(tricky_question)
          JamesBond.should have_received("Well done James!")
        end
      end
    end

## Contract tests

Contract tests are an idea, that whenever we stub something with a meaningful return value, we should add a test for the stubbed object. 

I’m sure you’re wondering about this part:

    verify_contract(:james_bond)

This line verifies that the stubbed methods are also tested by you. It reminds you to write tests for them and through that it helps you to put together those integration points between collaborators. Bogus won’t be able to write the tests for you, but reminding you to do so provides a lot of value and helps you to keep track of your temporary collaborators. 

Whenever you use named fakes like

    fake(:james_bond)

Bogus will remember any interactions you set up on that fake. If you want to verify that you tested all the scenarios specified by stubbing/spying/mocking on the fake object –

    verify_contract(:fake_name)

– is your friend.

We’ll dive more into test-doubles with Bogus in the next video.

***Stay shipping!***
