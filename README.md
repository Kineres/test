package customermanager;


import java.io.*;
import java.util.*;

public class CustomerBSTLab {

    // ==================== 1. CUSTOMER ====================
    static class Customer {
        int ID;
        String Name;
        String Gender;
        int Age;
        String Blood;

        Customer(int ID, String Name, String Gender,
                 int Age, String Blood) {
            this.ID = ID;
            this.Name = Name;
            this.Gender = Gender;
            this.Age = Age;
            this.Blood = Blood;
        }
    }

    // ==================== 2. NODE ====================
    static class Node {
        Customer data;
        Node left, right;

        Node(Customer data) {
            this.data = data;
        }
    }

    static Node root = null;
    static final String FILE_NAME = "customers.txt";
    static Scanner sc = new Scanner(System.in);

    // ==================== CAU 1: INSERT ====================
    static Node insert(Node root, Customer c) {
        if (root == null) {
            return new Node(c);
        }

        if (c.ID < root.data.ID) {
            root.left = insert(root.left, c);
        } else if (c.ID > root.data.ID) {
            root.right = insert(root.right, c);
        } else {
            System.out.println("ID da ton tai!");
        }

        return root;
    }

    // Duyet LNR: trai - goc - phai
    // Lay danh sach tang dan theo ID
    static void inorder(Node root, ArrayList<Customer> list) {
        if (root == null) return;

        inorder(root.left, list);
        list.add(root.data);
        inorder(root.right, list);
    }

    static ArrayList<Customer> getList() {
        ArrayList<Customer> list = new ArrayList<>();
        inorder(root, list);
        return list;
    }

    // ==================== CAU 2: DISPLAY TABLE ====================
    static void display(ArrayList<Customer> list) {
        if (list.isEmpty()) {
            System.out.println("Danh sach rong!");
            return;
        }

        System.out.printf("%-6s %-25s %-10s %-6s %-8s%n",
                "ID", "Name", "Gender", "Age", "Blood");
        System.out.println(
                "---------------------------------------------------------------");

        for (Customer c : list) {
            System.out.printf("%-6d %-25s %-10s %-6d %-8s%n",
                    c.ID, c.Name, c.Gender, c.Age, c.Blood);
        }
    }

    // ==================== CAU 3: AGE STATISTICS ====================
    static void ageStatistics() {
        ArrayList<Customer> list = getList();

        if (list.isEmpty()) {
            System.out.println("Danh sach rong!");
            return;
        }

        int sum = 0;
        Customer max = list.get(0);
        Customer min = list.get(0);

        for (Customer c : list) {
            sum += c.Age;

            if (c.Age > max.Age) max = c;
            if (c.Age < min.Age) min = c;
        }

        System.out.printf("Tuoi trung binh: %.2f%n",
                (double) sum / list.size());

        System.out.println("Khach hang lon tuoi nhat:");
        displayOne(max);

        System.out.println("Khach hang nho tuoi nhat:");
        displayOne(min);
    }

    static void displayOne(Customer c) {
        System.out.printf("%d | %s | %s | %d | %s%n",
                c.ID, c.Name, c.Gender, c.Age, c.Blood);
    }

    // ==================== CAU 4: FILTER ====================
    static ArrayList<Customer> filter(String field, String value) {
        ArrayList<Customer> result = new ArrayList<>();

        for (Customer c : getList()) {
            if (field.equals("Gender")
                    && c.Gender.equalsIgnoreCase(value)) {
                result.add(c);
            }

            if (field.equals("Blood")
                    && c.Blood.equalsIgnoreCase(value)) {
                result.add(c);
            }
        }

        return result;
    }

    // ==================== CAU 5: SORT MALE BY NAME ====================
    static void sortMaleByName() {
        ArrayList<Customer> males = filter("Gender", "Nam");

        Collections.sort(males, new Comparator<Customer>() {
            @Override
            public int compare(Customer c1, Customer c2) {
                return c1.Name.compareToIgnoreCase(c2.Name);
            }
        });

        display(males);
    }

    // ==================== CAU 6: MALE OLDEST / YOUNGEST ====================
    static void maleAgeStatistics() {
        ArrayList<Customer> males = filter("Gender", "Nam");

        if (males.isEmpty()) {
            System.out.println("Khong co khach hang nam!");
            return;
        }

        Customer max = males.get(0);
        Customer min = males.get(0);

        for (Customer c : males) {
            if (c.Age > max.Age) max = c;
            if (c.Age < min.Age) min = c;
        }

        System.out.println("Nam lon tuoi nhat:");
        displayOne(max);

        System.out.println("Nam nho tuoi nhat:");
        displayOne(min);
    }

    // ==================== CAU 7: SEARCH BY ID ====================
    static Customer search(Node root, int ID) {
        if (root == null) return null;

        if (ID == root.data.ID) {
            return root.data;
        }

        if (ID < root.data.ID) {
            return search(root.left, ID);
        }

        return search(root.right, ID);
    }

    // ==================== CAU 8: DELETE BY ID ====================
    static Node findMin(Node root) {
        while (root.left != null) {
            root = root.left;
        }
        return root;
    }

    static Node delete(Node root, int ID) {
        if (root == null) return null;

        if (ID < root.data.ID) {
            root.left = delete(root.left, ID);
        } else if (ID > root.data.ID) {
            root.right = delete(root.right, ID);
        } else {
            // Truong hop 1: khong co con trai
            if (root.left == null) return root.right;

            // Truong hop 2: khong co con phai
            if (root.right == null) return root.left;

            // Truong hop 3: co ca hai con
            Node min = findMin(root.right);
            root.data = min.data;
            root.right = delete(root.right, min.data.ID);
        }

        return root;
    }

    // ==================== CAU 9: SAVE FILE ====================
    static void saveFile() {
        try (PrintWriter pw = new PrintWriter(
                new FileWriter(FILE_NAME))) {

            for (Customer c : getList()) {
                pw.println(c.ID + "|" + c.Name + "|"
                        + c.Gender + "|" + c.Age + "|" + c.Blood);
            }

            System.out.println("Da luu file " + FILE_NAME);
        } catch (IOException e) {
            System.out.println("Loi ghi file: " + e.getMessage());
        }
    }

    // Doc file va nap vao BST
    static void loadFile() {
        File file = new File(FILE_NAME);

        if (!file.exists()) {
            System.out.println("Chua co file, bat dau danh sach moi.");
            return;
        }

        try (BufferedReader br = new BufferedReader(
                new FileReader(file))) {

            String line;

            while ((line = br.readLine()) != null) {
                try {
                    String[] a = line.split("\\|", -1);

                    if (a.length != 5) continue;

                    int ID = Integer.parseInt(a[0]);
                    String Name = a[1];
                    String Gender = a[2];
                    int Age = Integer.parseInt(a[3]);
                    String Blood = a[4];

                    root = insert(root,
                            new Customer(ID, Name, Gender, Age, Blood));
                } catch (NumberFormatException e) {
                    System.out.println("Bo qua dong loi: " + line);
                }
            }

            System.out.println("Da doc du lieu tu file.");
        } catch (IOException e) {
            System.out.println("Loi doc file: " + e.getMessage());
        }
    }

    // ==================== INPUT ====================
    static int readInt(String message) {
        while (true) {
            try {
                System.out.print(message);
                return Integer.parseInt(sc.nextLine().trim());
            } catch (NumberFormatException e) {
                System.out.println("Vui long nhap so nguyen!");
            }
        }
    }

    static Customer inputCustomer() {
        int ID = readInt("Nhap ID: ");

        System.out.print("Nhap ho va ten: ");
        String Name = sc.nextLine().trim();

        System.out.print("Nhap gioi tinh (Nam/Nu): ");
        String Gender = sc.nextLine().trim();

        int Age = readInt("Nhap tuoi: ");

        System.out.print("Nhap nhom mau (A/B/O): ");
        String Blood = sc.nextLine().trim().toUpperCase();

        return new Customer(ID, Name, Gender, Age, Blood);
    }

    // ==================== MAIN MENU ====================
    public static void main(String[] args) {
        loadFile();

        while (true) {
            System.out.println("\n========== CUSTOMER BST ==========");
            System.out.println("1. Them khach hang");
            System.out.println("2. Xuat danh sach");
            System.out.println("3. Tuoi trung binh, lon nhat, nho nhat");
            System.out.println("4. Loc Nam/Nu va nhom mau A/B/O");
            System.out.println("5. Sap xep khach hang nam theo ten");
            System.out.println("6. Nam lon tuoi nhat, nho tuoi nhat");
            System.out.println("7. Tim khach hang theo ID");
            System.out.println("8. Xoa khach hang theo ID");
            System.out.println("9. Luu file");
            System.out.println("0. Luu va thoat");

            int choice = readInt("Chon: ");

            switch (choice) {
                case 1: {
                    Customer c = inputCustomer();

                    if (search(root, c.ID) != null) {
                        System.out.println("ID da ton tai!");
                    } else {
                        root = insert(root, c);
                        saveFile();
                    }
                    break;
                }

                case 2:
                    display(getList());
                    break;

                case 3:
                    ageStatistics();
                    break;

                case 4: {
                    System.out.println("1. Nam");
                    System.out.println("2. Nu");
                    System.out.println("3. Nhom mau A");
                    System.out.println("4. Nhom mau B");
                    System.out.println("5. Nhom mau O");

                    int sub = readInt("Chon: ");

                    if (sub == 1) display(filter("Gender", "Nam"));
                    else if (sub == 2) display(filter("Gender", "Nu"));
                    else if (sub == 3) display(filter("Blood", "A"));
                    else if (sub == 4) display(filter("Blood", "B"));
                    else if (sub == 5) display(filter("Blood", "O"));
                    else System.out.println("Lua chon khong hop le!");
                    break;
                }

                case 5:
                    sortMaleByName();
                    break;

                case 6:
                    maleAgeStatistics();
                    break;

                case 7: {
                    int ID = readInt("Nhap ID can tim: ");
                    Customer c = search(root, ID);

                    if (c == null) {
                        System.out.println("Khong tim thay!");
                    } else {
                        displayOne(c);
                    }
                    break;
                }

                case 8: {
                    int ID = readInt("Nhap ID can xoa: ");

                    if (search(root, ID) == null) {
                        System.out.println("Khong tim thay ID!");
                    } else {
                        root = delete(root, ID);
                        saveFile();
                        System.out.println("Da xoa khach hang!");
                    }
                    break;
                }

                case 9:
                    saveFile();
                    break;

                case 0:
                    saveFile();
                    System.out.println("Ket thuc chuong trinh.");
                    sc.close();
                    return;

                default:
                    System.out.println("Lua chon khong hop le!");
            }
        }
    }
}
