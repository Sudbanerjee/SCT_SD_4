# SCT_SD_4 
#include <iostream>
#include <fstream>
#include <vector>
#include <string>
#include <curl/curl.h>
#include "pugixml.hpp"

// Function to write data received by libcurl
size_t WriteCallback(void* contents, size_t size, size_t nmemb, std::string* s) {
    s->append((char*)contents, size * nmemb);
    return size * nmemb;
}

// Function to fetch HTML content from a URL
std::string fetchHTML(const std::string& url) {
    CURL* curl;
    CURLcode res;
    std::string readBuffer;

    curl = curl_easy_init();
    if (curl) {
        curl_easy_setopt(curl, CURLOPT_URL, url.c_str());
        curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, WriteCallback);
        curl_easy_setopt(curl, CURLOPT_WRITEDATA, &readBuffer);
        res = curl_easy_perform(curl);
        curl_easy_cleanup(curl);
    }
    return readBuffer;
}

// Function to parse HTML content and extract product information
std::vector<std::vector<std::string>> parseHTML(const std::string& html) {
    std::vector<std::vector<std::string>> productData;

    pugi::xml_document doc;
    pugi::xml_parse_result result = doc.load_string(html.c_str());

    if (result) {
        // Assuming products are contained within <div class="product-item">
        pugi::xpath_node_set products = doc.select_nodes("//div[@class='product-item']");

        for (auto product : products) {
            std::vector<std::string> productInfo;
            std::string name = product.node().child("h2").child_value("product-name");
            std::string price = product.node().child("span").child_value("product-price");
            std::string rating = product.node().child("span").child_value("product-rating");

            productInfo.push_back(name);
            productInfo.push_back(price);
            productInfo.push_back(rating);

            productData.push_back(productInfo);
        }
    }

    return productData;
}

// Function to write product data to a CSV file
void writeCSV(const std::string& filename, const std::vector<std::vector<std::string>>& data) {
    std::ofstream file(filename);

    // Write the header
    file << "Name,Price,Rating\n";

    // Write the data
    for (const auto& product : data) {
        file << product[0] << "," << product[1] << "," << product[2] << "\n";
    }

    file.close();
}

int main() {
    std::string url = "http://example.com/products";
    std::string htmlContent = fetchHTML(url);

    std::vector<std::vector<std::string>> productData = parseHTML(htmlContent);

    writeCSV("products.csv", productData);

    std::cout << "Product information extracted and saved to products.csv" << std::endl;

    return 0;
}


