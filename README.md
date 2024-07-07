# -PRODIGY_SD_05-
coding a program that extracts product information
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class ProductScraper {

    public static void main(String[] args) {
        String url = "https://www.amazon.com/best-sellers-books-Amazon/zgbs/books/";

        try {
            List<Product> products = scrapeProducts(url);

            if (products != null && !products.isEmpty()) {
                writeToCSV(products, "products.csv");
                System.out.println("Product data extracted and saved to products.csv");
            } else {
                System.out.println("No product data found.");
            }
        } catch (IOException e) {
            System.err.println("Error scraping product data: " + e.getMessage());
        }
    }

    // Method to scrape product information from Amazon
    public static List<Product> scrapeProducts(String url) throws IOException {
        List<Product> products = new ArrayList<>();

        Document doc = Jsoup.connect(url).get();
        Elements productElements = doc.select("div.zg_itemImmersion");

        for (Element productElement : productElements) {
            String name = productElement.select("div.p13n-sc-truncate").text().trim();
            String price = productElement.select("span.p13n-sc-price").text().trim();
            String rating = productElement.select("span.a-icon-alt").text().trim();

            // Remove unnecessary text around the rating
            if (rating.contains(" out of 5 stars")) {
                rating = rating.replace(" out of 5 stars", "");
            }

            products.add(new Product(name, price, rating));
        }

        return products;
    }

    // Method to write product data to CSV file
    public static void writeToCSV(List<Product> products, String fileName) {
        try (FileWriter writer = new FileWriter(fileName)) {
            writer.append("Name,Price,Rating\n");
            for (Product product : products) {
                writer.append(String.join(",", product.getName(), product.getPrice(), product.getRating()))
                        .append("\n");
            }
        } catch (IOException e) {
            System.err.println("Error writing to CSV file: " + e.getMessage());
        }
    }

    // Product class representing a product with name, price, and rating
    static class Product {
        private String name;
        private String price;
        private String rating;

        public Product(String name, String price, String rating) {
            this.name = name;
            this.price = price;
            this.rating = rating;
        }

        public String getName() {
            return name;
        }

        public String getPrice() {
            return price;
        }

        public String getRating() {
            return rating;
        }
    }
}
