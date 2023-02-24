import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Scanner;
import javax.swing.JOptionPane;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int numLinks = askNumLinks(scanner);
        for (int i = 1; i <= numLinks; i++) {
            setEthernetLink(i);
        }
        for (int i = 1; i <= numLinks; i++) {
           
            askAuthType(scanner, numLinks, i);
            
        }
        runMikrotikScript("/ip dns set servers=8.8.8.8");
        runMikrotikScript("/ip firewall nat add action=masquerade chain=srcnat");
        runMikrotikScript("////////////////////////////////////////////////////////////////////");
        exibirResultadoFinal(numLinks);
        
 
    }
    

   private static int askNumLinks(Scanner scanner) {
    String response = JOptionPane.showInputDialog(null, "Quantos links serão usados?");
    return Integer.parseInt(response);
}

private static String askIpAddress(Scanner scanner, int linkNum) {
    String response = JOptionPane.showInputDialog(null, "Qual IP para a porta " + linkNum + "?");
    return response;
}

private static String askGateway(Scanner scanner, int linkNum) {
    String response = JOptionPane.showInputDialog(null, "Qual é o gateway para a porta " + linkNum + "?");
    return response;
}

   private static String askAuthType(Scanner scanner, int numLinks, int i) {
  
       String[] options = { "PPPoE", "DHCP", "IP Fixo" };
    int selectedOption = JOptionPane.showOptionDialog(null, "Qual tipo de autenticação para a porta " + i + "?", "Escolha o tipo de autenticação", JOptionPane.DEFAULT_OPTION, JOptionPane.QUESTION_MESSAGE, null, options, options[0]);
   String gateway;
    switch (selectedOption) {
        case 0:
            String user = JOptionPane.showInputDialog(null, "Qual usurio pppoe " + i + "?");
            String password = JOptionPane.showInputDialog(null, "Qual password pppoe " + i + "?");
            
            editarPPPoE(i, password, user);
            
             gateway = askGateway(scanner, i);
            addIpRoute(i, gateway,numLinks);
            addFirewallMangle(i,numLinks);
            
            return "pppoe";
        case 1:
            
            addDhcpClient(i);
            gateway = askGateway(scanner, i);
            addIpRoute(i, gateway,numLinks);
            addFirewallMangle(i,numLinks);
            
            return "dhcp-client";
        case 2:
            
             String ipAddress = askIpAddress(scanner, i);
             gateway = askGateway(scanner, i);
            addIpAddress(ipAddress, i);
            addIpRoute(i, gateway,numLinks);
            addFirewallMangle(i,numLinks);
          
            return "static";
        default:
            return "";
    }
}
   
   

   
private static void addDhcpClient(int linkNum) {
    String command = "/ip dhcp-client\n" +
"add add-default-route=no disabled=no interface=\"LINK "+linkNum+"\" use-peer-dns=\\\n" +
"    no";
    runMikrotikScript(command);
}

    private static void setEthernetLink(int linkNum) {
        String command = "/interface ethernet set [ find default-name=ether"
                + linkNum + " ] name=\"LINK " + linkNum + "\"";
        runMikrotikScript(command);
    }


    
    
    private static void addIpAddress(String ipAddress, int linkNum) {
        String command = "/ip address add address=" + ipAddress + 
                " interface=\"LINK " + linkNum + "\" network=" 
                + getNetwork(ipAddress) + ".0";
        runMikrotikScript(command);
    }

    private static void addIpRoute(int linkNum, String gateway,int numLinks) {
        int num =numLinks + linkNum;
        String command = "/ip route add distance=" + linkNum + " gateway="
                + gateway + " routing-mark=\"ROTA LINK " + linkNum + "\"";
        runMikrotikScript(command);
        command = "/ip route add distance="+num+" gateway=" + gateway;
        runMikrotikScript(command);
    }

    private static void addFirewallMangle(int linkNum ,int Links) {
        int Num=linkNum-1;
       
        String command = "/ip firewall mangle add action=mark-routing chain=prerouting dst-address-type=!local new-routing-mark=\"ROTA LINK " + linkNum + "\" passthrough=yes per-connection-classifier=both-addresses:" +Links + "/"+ Num ;
        runMikrotikScript(command);
    }

    private static String getNetwork(String ipAddress) {
        String[] octets = ipAddress.split("\\.");
        return octets[0] + "." + octets[1] + "." + octets[2];
    }
    
   
    

    private static void runMikrotikScript(String text) {
        try {
            String filePath = System.getProperty("user.dir") + "/resultados.txt";
            FileWriter writer = new FileWriter(filePath, true);
            BufferedWriter bufferedWriter = new BufferedWriter(writer);
            bufferedWriter.write(text);
            bufferedWriter.newLine();
            bufferedWriter.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    

    

    public static void editarPPPoE(int linkNum, String password, String user) {
        String command = "/interface pppoe-client add add-default-route=yes disabled=no interface=\"LINK "+linkNum+"\" name=\\ \"PPPOE LINK "+linkNum+"\" password="+password+" user="+user;
        runMikrotikScript(command);
    }





    private static void exibirResultadoFinal(int numLinks) {
        System.out.println("Configuração concluída para " + numLinks + " links.");
    }
}
