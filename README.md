# UI_WebScraping
Extract any email or phone number from a html page

Form1.cs

using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Data.SqlClient;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using HtmlAgilityPack;
using TheLibrary;

namespace WebScrapingForContactInfosForm
{
    public partial class Form1 : Form
    {

        string connectionString = "Data Source=.;Initial Catalog=ContactInfoDB;Integrated Security=True;";
        List<string> phoneNumbers = new List<string>();
        List<string> emails = new List<string>();
        public Form1()
        {
            InitializeComponent();
            textBox1.Text = "http://vanceamihaicatalin.com";
        }

        private void button1_Click(object sender, EventArgs e)
        {
            string url = textBox1.Text;
            var web = new HtmlWeb();
            var htmlDoc = web.Load(url);
            richTextBox1.Text = htmlDoc.Text;
            if (htmlDoc.DocumentNode != null)
            {
                foreach (HtmlNode node in htmlDoc.DocumentNode.SelectNodes("//a[starts-with(@href, 'mailto:')]/text()"))
                {
                    EmailClass emailClass = new EmailClass(node.InnerText);
                    emails.Add(emailClass.EmailString);
                    richTextBox2.Lines = emails.ToArray();

                }

                foreach (HtmlNode node in htmlDoc.DocumentNode.SelectNodes("//a[starts-with(@href, 'tel:')]/text()"))
                {
                    PhoneClass phoneClass = new PhoneClass(node.InnerText);
                    phoneNumbers.Add(phoneClass.PhoneString);
                    richTextBox3.Lines = phoneNumbers.ToArray();
                }
            }            
        }
        // Eroare linia 67: System.Data.SqlClient.SqlException: 'Invalid object name 'emailsTable'.'
        // Eroare linia 82: System.Data.SqlClient.SqlException: 'Invalid object name 'phonesTable'.'

        private void button2_Click(object sender, EventArgs e)
        {
            using (SqlConnection connection = new SqlConnection(connectionString))
            {
                connection.Open();

                foreach (string email in emails)
                {
                    //SqlCommand myCommand = new SqlCommand();
                    //myCommand.CommandText=(@"INSERT INTO emailsTable(emailString) VALUES ('"+ email +"')");
                    //myCommand.ExecuteNonQuery();
                    SqlDataAdapter myCommand = new SqlDataAdapter();
                    myCommand.InsertCommand = new SqlCommand(@"INSERT INTO emailsTable(emailString) VALUES ('" + email + "')");
                    myCommand.InsertCommand.Connection = connection;
                    myCommand.InsertCommand.ExecuteNonQuery();

                }
                connection.Close();
                richTextBox2.Clear();
            }
        }

        private void button3_Click(object sender, EventArgs e)
        {
            using (SqlConnection connection = new SqlConnection(connectionString))
            {
                connection.Open();

                foreach (string phoneNum in phoneNumbers)
                {
                    SqlDataAdapter myCommand = new SqlDataAdapter();
                    myCommand.InsertCommand = new SqlCommand(@"INSERT INTO phonesTable(phoneNumber) VALUES ('" + phoneNum + "')");
                    myCommand.InsertCommand.Connection = connection;
                    myCommand.InsertCommand.ExecuteNonQuery();

                }
                connection.Close();
                richTextBox3.Clear();

            }
        }
    }
}
