# pykafka

pykafka allows you to produce messages to the Kafka distributed publish/subscribe messaging service.

## Requirements

You need to have access to your Kafka instance and be able to connect through
TCP. You can obtain a copy and instructions on how to setup kafka at
https://github.com/kafka-dev/kafka

## Installation
easy_install -f 'https://github.com/DataDog/pykafka/tarball/2.1.0#egg=pykafka-2.1.0' pykafka

## Usage

### Sending a simple message

    import kafka
    kafka = kafka.Kafka(host='localhost')
    kafka.produce("test-topic", "Hello World")

### Sending a sequence of messages

    import kafka
    kafka = kafka.Kafka(host='localhost')
    kafka.produce("test-topic", ["Hello", "World"])

### Consuming messages one by one

    import kafka
    kafka = kafka.Kafka(host='localhost')
    for offset, message in kafka.fetch("test-topic", offset=0):
        print message

### Nonblocking Tornado client support

    import time
    import tornado.ioloop
    import tornado.web

    from kafka import LATEST_OFFSET
    from kafka.nonblocking import KafkaTornado

    class MainHandler(tornado.web.RequestHandler):
        def initialize(self, kafka, topic):
            self.kafka = kafka
            self.topic = topic
    
        def post(self):
            data = self.get_argument('data')
            self.kafka.produce(self.topic, data)
        
        @tornado.web.asynchronous
        def get(self):
            kafka.offsets(self.topic, LATEST_OFFSET, max_offsets=2, 
                callback=self._on_offset)
    
        def _on_offset(self, offsets):
            offset = offsets[-1] # Get the second to latest offset
            kafka.fetch(self.topic, offset, callback=self._on_fetch)
    
        def _on_fetch(self, messages):
            for offset, message in messages:
                self.write("{0}: {1}".format(offset, message))
            self.finish()
    

    kafka = KafkaTornado()

    application = tornado.web.Application([
        (r"/", MainHandler, {
            'kafka': kafka,
            'topic': 'hello-world'
        }),
    ])

    if __name__ == "__main__":
        parse_command_line()
        application.listen(8888)
        tornado.ioloop.IOLoop.instance().start()

    

Contact:

Please use the GitHub issues: https://github.com/datadog/pykafka/issues

* Forked from https://github.com/dsully/pykafka which was inspired by Alejandro Crosa's kafka-rb: https://github.com/acrosa/kafka-rb
