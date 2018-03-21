
import java.util.Hashtable;

import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.Destination;
import javax.jms.ExceptionListener;
import javax.jms.JMSException;
import javax.jms.MessageConsumer;
import javax.jms.MessageProducer;
import javax.jms.Session;
import javax.jms.TextMessage;
import javax.naming.Context;
import javax.naming.InitialContext;

import org.apache.log4j.Logger;

import com.microsoft.azure.servicebus.primitives.ConnectionStringBuilder;


public class JmsQueueSend {
	
	static final String SB_SAMPLES_CONNECTIONSTRING = "Endpoint=XXXX";
	
	 private static Logger logger= Logger.getRootLogger();
	 
	// 400Bytes
	private String MESSAGE = "ABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDasdABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDasdABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABaqwCDABCDABCDABCDABCDABCDABCDasdABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDABCDasda";

	 public void run(String connectionString) throws Exception {

	        ConnectionStringBuilder csb = new ConnectionStringBuilder(connectionString);
	        
	        // set up JNDI context
	        Hashtable<String, String> hashtable = new Hashtable<>();
	        hashtable.put("connectionfactory.SBCF", "failover:(amqps://" + csb.getEndpoint().getHost() + ")?failover.reconnectDelay=2000&failover.maxReconnectAttempts=-1&jms.forceAsyncSend=true");
	        hashtable.put("queue.QUEUE", "test");
	        hashtable.put(Context.INITIAL_CONTEXT_FACTORY, "org.apache.qpid.jms.jndi.JmsInitialContextFactory");
	        Context context = new InitialContext(hashtable);
	        ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCF");
	        
	        // Look up queue
	        Destination queue = (Destination) context.lookup("QUEUE");

	            // Create Connection
	            Connection connection = cf.createConnection(csb.getSasKeyName(), csb.getSasKey());
	            ExceptionListener listener = new ExceptionListener() {
					
					@Override
					public void onException(JMSException exception) {
						System.err.println("exec msg :: " + exception.getMessage());
						exception.printStackTrace();
					}
				};
				connection.setExceptionListener(listener);
	            connection.start();
	            
	            // Create Session, no transaction, client ack
	            Session session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);

	            MessageProducer messageProducer = session.createProducer(queue);
	            
	            TextMessage message = session.createTextMessage(MESSAGE);    
	           
	            logger.info("*** Producer Started ***");
	            for(int i=0; i <10000; i++){
	            	messageProducer.send(message);
	            }
	            logger.info("*** Producer end ***");
	            
	            System.out.println("*** Producer created ***");
	            
	            
	    }

	    public static void main(String[] args) throws Exception {
	    	
	    	JmsQueueSend app = new JmsQueueSend();
	    	 app.run(SB_SAMPLES_CONNECTIONSTRING);
	    	 
	    }
	    
}
