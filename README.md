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
            using (SqlConnection connection = new SqlConnection(connectionString))
            {
                connection.Open();
                if (htmlDoc.DocumentNode != null)
                {
                    foreach (HtmlNode node in htmlDoc.DocumentNode.SelectNodes("//a[starts-with(@href, 'mailto:')]/text()"))
                    {
                        EmailClass emailClass = new EmailClass(node.InnerText);
                        emails.Add(emailClass.EmailString);
                        richTextBox2.Lines = emails.ToArray();
                        SqlDataAdapter myCommand = new SqlDataAdapter();
                        myCommand.InsertCommand = new SqlCommand(@"INSERT INTO emailsTable(emailString) VALUES ('" + emailClass.EmailString + "')");
                        myCommand.InsertCommand.Connection = connection;
                        myCommand.InsertCommand.ExecuteNonQuery();

                    }

                    foreach (HtmlNode node in htmlDoc.DocumentNode.SelectNodes("//a[starts-with(@href, 'tel:')]/text()"))
                    {
                        PhoneClass phoneClass = new PhoneClass(node.InnerText);
                        phoneNumbers.Add(phoneClass.PhoneString);
                        richTextBox3.Lines = phoneNumbers.ToArray();
                        SqlDataAdapter myCommand = new SqlDataAdapter();
                        myCommand.InsertCommand = new SqlCommand(@"INSERT INTO phonesTable(phoneNumber) VALUES ('" + phoneClass.PhoneString + "')");
                        myCommand.InsertCommand.Connection = connection;
                        myCommand.InsertCommand.ExecuteNonQuery();
                    }
                    MessageBox.Show("Info saved !!!");
                    connection.Close();
                }
                else MessageBox.Show("Can't save those infos");
                
            }
        }

    }
}
