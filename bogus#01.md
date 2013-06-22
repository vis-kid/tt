Bogus

Bogus is a library that aims to reduce the risks associated with isolated unit testing.

it "checks protocol for 007’s mission start" do
  briefing = stub( meeting_with_M: [top_secret_briefing]) 
	gadgets  = stub( meeting_with_Q: [stylish_Aston_Martin, laser_Rolex])

	mission_start = Mission.new(briefing, gadgets)
	mission_start.check_protocol.should == mission_is_a_go
end

Problem:
It can be summarized by “Programming by wishful thinking.”
Classes under test not only know how methods work
-> they also specify how tested classes will interact with it’s collaborators
-> and what those collaborators will be

This top-down system design, where classes get implemented before other collaborators exist yet, seems to be a very good thing in principle. You only implement what you need.

BUT!
The same freedom from not having to implement collaborators can quickly turn on you.

Once the tested class is implemented – what tells you that the collaborators aren’t implemented yet?
In Plain English:
This kind of stabbing requires you to write integration or end-to-end tests => you have to make sure the pieces fit together.

The problem with those integration tests is that they unnecessarily add slower and harder to set up tests to cover all the integration points between the objects/collaborators.

Another potential problem lurks in the dark because those stubbed collaborators will likely be interacting with more objects than the tested class. => which opens the door wide open for duplication. You know what awaits you if you wanna change the collaborators later on... Remember all the places where you created that stub? Your tests won’t help you – they don’t have a clue what the collaborators interface should look like.

THE SOLUTION:
Bogus makes your test doubles more ASSERTIVE!
They will tell you “Hey, you are stubbing the wrong thing here!”

*It does not only make sure the stubbed class work in your tests
*It also assures those stubbed collaborators classes EXIST and have an INTERFACE.

The syntax is quite simple and similar:
stub(meeting_with_M: [top_secret_briefing])
=>
fake(:M, meeting_with_M: [top_secret_briefing])
=> faked class M exits now and has an interface through #meeting_with_M and returns [top_secret_briefing]

To test the mission_start you need 3 classes – 2 of them stubbed out:
Mission, M and Q

it "checks protocol for 007’s mission start" do
  briefing = fake(:M, meeting_with_M: [top_secret_briefing])
	gadgets  = fake(:Q, meeting_with_Q: [stylish_Aston_Martin, laser_Rolex])

	mission_start = Mission.new(briefing, gadgets)
	mission_start.check_protocol.should == mission_is_a_go
end

Very similar syntax that you’re used to but adds a lot of goodies.
The question arises of course, “How much DRYer is this solution really?”
???
The documentation says it’s slightly better for drying up your code. You only  need to ever stub methods that return meaningful values.
???

The neat thing you can do to DRY your stubs pulling them out in a fakes.rb file in spec/support/fakes.rb
You can put methods that return values in this fakes file. All you need to do is provide a reasonable default return value for those methods.

The syntax looks like this:
Bogus.fakes do
  fake(:m) do
    drop_a_random_lexical_question_on_007 ["Well done James!"]
	end

  fake(:james_bond) do
	  asked_random_lexical_question [witty_answer]
	end
end

In the test you only have to use it like this:

describe MissionBriefing do
  let(:witty_question) { "How many agents were killed in the James Bond series?" }

  describe M do
		fake(:james_bond)

		it "drops random lexical questions on 007 "
      JamesBond.asked_lexical_question(witty_question)			
			M.should have_received(witty_answer)
		end

	describe JamesBond do
		fake(:m)
		verify_contract(:james_bond)

		it "impress M with random knowledge bombs" do
			M.drop_a_random_lexical_question_on_007(witty_question)
			JamesBond.should have_received("Well done James!")
		end
	end
end








