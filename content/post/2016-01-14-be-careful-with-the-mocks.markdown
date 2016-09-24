---
author: Federico Grosso
categories: tests mocks
comments: true
date: 2016-01-14T15:13:41Z
title: Be careful with the mocks
url: /2016/01/14/be-careful-with-the-mocks/
---

Mocking objects is a common practice when writing tests, however it can be painful when refactoring a class tested with mocks. I will show a simple example to explain the problems that mocking can generate, but first I would like to give some definitions:

- **Stubs** are objects that respond to a subset of messages from the object to be stubbed with specific responses.
- **Mocks** are objects that verify the integration between the system under test and the mocked object.
- **Fake** are objects with implementation that simulate the real one but they are simpler and/or faster.

<!--more-->

Suppose we have the next class for sending email notifications to an user of our system:

{{< highlight ruby >}}
class EmailNotifier {
    
    def initialize(email_client, from_email) {
        @email_client = email_client
        @from_email = from_email
    }

    def notify(an_user, a_message) {
        @email_client.send(@from_email, an_user.email, a_message)
    }

}
{{< / highlight >}}


There are some different ways to write the tests for the `notify` method. Maybe the easiest way of doing it is using mocks:

{{< highlight ruby >}}
describe EmailNotification do

    describe :notify do    

        it 'sends an email to the given user with the given message' do
            
            from_email = "server@test.com"
            user_email_address = "user@test.com"
            user = double("User", email: user_email_address)
            message = "Message"

            email_client = double('EmailClient')
            expect(email_client).to receive(:send).with(from_email, user_email_address, message).and_return(:success)

            notifier = EmailNotifier.new email_client, from_email
            notifier.notify(user, message)
        end
        
    end
end
{{< / highlight >}}

On this test we have created one stub for User and a mock for EmailClient. In order to check if an email is sent, we defined the expectations knowing the implementation should call the send message, but by doing this we are writing tests that are strongly coupled to their method implementation because it is expecting `notify` to send the message `send` to `email_client`. So, in my opinion, testing a message in this way has the next problems:

- **It is testing HOW the method is implemented instead of WHAT the method should do**, which may result on test failures if we change the code. Instead of testing the `send` message is called, we should be testing an email has been sent.
- **It forces you to write the expected behavior of the mocked class in each test**, which distracts you from the test itself when reading and writing the test, and also, it may increase the difficulty of understanding it.
- **The tests are hard to maintain**, changing a message of the mocked class will require to change all the places in which that message was mocked. If one of the tests that mocks this message is not changed, it may generate a *false positive*, for example, if you remove the message `send` from *EmailClient*, the mock wouldn't fail, so the test will pass.

One option to solve the first 2 problems and reduce the impact of the last one can be to use a *Fake object*. In this case, we should write a *FakeEmailClient*, which should respond to the same messages of the real *EmailNotifier*, and also it should have a message to test the sender, the receiver, the message and if an email has been sent. Creating a *FakeEmailClient* won't solve the *false positive* problem, but it will be easier to fix because you should only update the fake object instead of all the tests where the *EmailClient* is used.

Let's assume the *EmailClient* class has a constructor without parameters, and it only has the `send(from, to, message)` method. In that case, the *Fake Class* would be:

{{< highlight ruby >}}
class FakeEmailClient {

    def send(sender_address, to_address, message) {
        @last_sender_address = from_address
        @last_receiver_address = to_address
        @last_message_sent = message
    }

    attr_reader :last_sender_address, :last_receiver_address, :last_message_sent
    
    def assert_last_email_received(from, to, message)
        expect(@last_sender_address).to eq(from_address)
        expect(@last_receiver_address).to eq(to_address)
        expect(@last_message_sent).to eq(message)
    end
    
{{< / highlight >}}

This *Fake object* should be used in every test that needs to use an *EmailClient*, and if you need to test if an email has been sent, you can ask it to the fake client.

{{< highlight ruby >}}
describe EmailNotification do

    describe :notify do    

        it 'sends an email to the given user with the given message' do
            
            from_email = "server@test.com"
            user_email_address = "user@test.com"
            user = double("User", email: user_email_address)
            message = "Message"

            email_client = FakeEmailClient.new

            notifier = EmailNotifier.new email_client, from_email
            notifier.notify(user, message)

            email_client.assert_last_email_received(from_email, user_email_address,message)
        end
        
    end
end
{{< / highlight >}}

To conclude, I am not saying that mocks are useless, but I think that avoiding them whenever is possible makes the tests safer and easier to understand and maintain.