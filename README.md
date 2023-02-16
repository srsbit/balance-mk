import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
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
            String ipAddress = askIpAddress(scanner, i);
            String gateway = askGateway(scanner, i);
            addIpAddress(ipAddress, i);
            addIpRoute(i, gateway,numLinks);
            addFirewallMangle(i,numLinks);
        }
        runMikrotikScript("/ip dns set servers=8.8.8.8");
        runMikrotikScript("/ip firewall nat\n" +
"add action=masquerade chain=srcnat");
        
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

   

    private static void setEthernetLink(int linkNum) {
        String command = "/interface ethernet set [ find default-name=ether"
                + linkNum + " ] name=\"LINK " + linkNum + "\"";
        runMikrotikScript(command);
    }

    private static void addIpAddress(String ipAddress, int linkNum) {
        String command = "/ip address add address=" + ipAddress + 
                "/24 interface=\"LINK " + linkNum + "\" network=" 
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

    private static void exibirResultadoFinal(int numLinks) {
        System.out.println("Configuração concluída para " + numLinks + " links.");
    }
}
