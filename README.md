import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Random;
import java.util.Scanner;

/**
 * Rule - one "intent": a set of keywords and possible replies.
 */
class Rule {
    private final List<String> keywords;
    private final List<String> responses;
    private final Random random = new Random();

    public Rule(String[] keywords, String[] responses) {
        this.keywords = Arrays.asList(keywords);
        this.responses = Arrays.asList(responses);
    }

    /** Score = how many of this rule's keywords appear in the input. */
    public int score(String normalizedInput) {
        int score = 0;
        for (String k : keywords) {
            if (normalizedInput.contains(" " + k + " ")) {
                score++;
            }
        }
        return score;
    }

    public String getResponse() {
        return responses.get(random.nextInt(responses.size()));
    }
}

/**
 * ResponseEngine - decides what the bot says.
 */
class ResponseEngine {
    private final List<Rule> rules = new ArrayList<>();
    private String userName = null;

    public ResponseEngine() {
        loadRules();
    }

    private void loadRules() {
        rules.add(new Rule(
            new String[]{"hello", "hi", "hey", "hii", "good morning", "good evening"},
            new String[]{"Hello! How can I help you today?",
                         "Hi there! What would you like to talk about?",
                         "Hey! Nice to see you."}));

        rules.add(new Rule(
            new String[]{"how are you", "how r u", "whats up", "how do you do"},
            new String[]{"I'm just a program, but I'm running great! How about you?",
                         "Doing well, thanks for asking!"}));

        rules.add(new Rule(
            new String[]{"your name", "who are you", "what are you"},
            new String[]{"I'm JavaBot, an AI chatbot built in Java as a course project.",
                         "Call me JavaBot! I was written in Java."}));

        rules.add(new Rule(
            new String[]{"java", "jvm", "jdk", "jre"},
            new String[]{"Java is a class-based, object-oriented language that runs on the JVM: write once, run anywhere.",
                         "The JDK has the tools to build Java programs, the JRE runs them, and the JVM executes the bytecode."}));

        rules.add(new Rule(
            new String[]{"oop", "object oriented", "object-oriented", "oops"},
            new String[]{"OOP has four pillars: encapsulation, inheritance, polymorphism and abstraction.",
                         "Object-Oriented Programming organizes code into classes and objects."}));

        rules.add(new Rule(
            new String[]{"ai", "artificial intelligence", "machine learning", "ml"},
            new String[]{"AI is the ability of machines to perform tasks that normally need human intelligence.",
                         "Machine learning is a branch of AI where systems learn patterns from data."}));

        rules.add(new Rule(
            new String[]{"chatbot", "bot", "chat bot"},
            new String[]{"A chatbot is a program that simulates conversation with users. I'm a rule-based one: I match keywords to replies.",
                         "Chatbots can be rule-based, NLP-based, or powered by large language models."}));
        rules.add(new Rule(
            new String[]{"help", "what can you do", "commands"},
            new String[]{"Try: greetings, 'what is java', 'what is oop', 'what is ai', 'time', 'date', 'my name is <name>', or 'bye'."}));

        rules.add(new Rule(
            new String[]{"thanks", "thank you", "thx"},
            new String[]{"You're welcome!", "Happy to help!", "Anytime!"}));

        rules.add(new Rule(
            new String[]{"joke", "funny"},
            new String[]{"Why do Java developers wear glasses? Because they don't C#!",
                         "There are 10 types of people: those who understand binary and those who don't."}));
    }

    /** Lowercase, strip punctuation, pad with spaces for whole-word matching. */
    private String normalize(String input) {
        String cleaned = input.toLowerCase().replaceAll("[^a-z0-9\\s-]", " ").replaceAll("\\s+", " ").trim();
        return " " + cleaned + " ";
    }

    public String getReply(String rawInput) {
        String normalized = normalize(rawInput);

        // 1. Special handlers (dynamic answers)
        if (normalized.contains(" my name is ")) {
            String after = normalized.substring(normalized.indexOf(" my name is ") + 12).trim();
            if (!after.isEmpty()) {
                userName = capitalize(after.split(" ")[0]);
                return "Nice to meet you, " + userName + "!";
            }
        }
        if (normalized.contains(" what is my name ") || normalized.contains(" do you know my name ")) {
            return userName != null ? "Your name is " + userName + "." : "You haven't told me your name yet.";
        }
        if (normalized.contains(" time ")) {
            return "The current time is " + LocalTime.now().format(DateTimeFormatter.ofPattern("hh:mm a")) + ".";
        }
        if (normalized.contains(" date ") || normalized.contains(" today ")) {
            return "Today's date is " + LocalDate.now().format(DateTimeFormatter.ofPattern("dd MMM yyyy")) + ".";
        }

        // 2. Rule matching: pick the rule with the highest keyword score
        Rule best = null;
        int bestScore = 0;
        for (Rule rule : rules) {
            int s = rule.score(normalized);
            if (s > bestScore) {
                bestScore = s;
                best = rule;
            }
        }
        if (best != null) {
            String reply = best.getResponse();
            return (userName != null && Math.random() < 0.3) ? userName + ", " + lowerFirst(reply) : reply;
        }

        // 3. Fallback
        return "Sorry, I didn't understand that. Type 'help' to see what I can talk about.";
    }

    private String capitalize(String s) {
        return s.substring(0, 1).toUpperCase() + s.substring(1);
    }

    private String lowerFirst(String s) {
        return s.substring(0, 1).toLowerCase() + s.substring(1);
    }
}

/**
 * ChatLogger - saves the conversation to a text file (file handling).
 */
class ChatLogger {
    private final String fileName;
    private final DateTimeFormatter fmt = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    public ChatLogger(String fileName) {
        this.fileName = fileName;
    }

    public void log(String speaker, String message) {
        try (PrintWriter out = new PrintWriter(new FileWriter(fileName, true))) {
            out.println("[" + LocalDateTime.now().format(fmt) + "] " + speaker + ": " + message);
        } catch (IOException e) {
            System.out.println("(Could not write to log file: " + e.getMessage() + ")");
        }
    }
}

/**
 * Main class - handles the conversation loop.
 */
public class ChatBot {
    public static void main(String[] args) {
        ResponseEngine engine = new ResponseEngine();
        ChatLogger logger = new ChatLogger("chat_log.txt");
        List<String> history = new ArrayList<>();

        System.out.println("=====================================");
        System.out.println("   JavaBot - Simple AI Chatbot");
        System.out.println("   Type 'help' for tips, 'bye' to exit");
        System.out.println("=====================================");

        try (Scanner scanner = new Scanner(System.in)) {
            while (true) {
                System.out.print("You : ");
                if (!scanner.hasNextLine()) break;
                String input = scanner.nextLine().trim();

                if (input.isEmpty()) {
                    System.out.println("Bot : Please type something.");
                    continue;
                }

                if (input.equalsIgnoreCase("bye") || input.equalsIgnoreCase("exit") || input.equalsIgnoreCase("quit")) {
                    String goodbye = "Goodbye! Have a great day!";
                    System.out.println("Bot : " + goodbye);
                    logger.log("User", input);
                    logger.log("Bot", goodbye);
                    break;
                }

                if (input.equalsIgnoreCase("history")) {
                    System.out.println("--- Conversation history ---");
                    history.forEach(System.out::println);
                    System.out.println("----------------------------");
                    continue;
                }

                String reply = engine.getReply(input);
                System.out.println("Bot : " + reply);

                history.add("You: " + input);
                history.add("Bot: " + reply);
                logger.log("User", input);
                logger.log("Bot", reply);
            }
        }
    }
}
