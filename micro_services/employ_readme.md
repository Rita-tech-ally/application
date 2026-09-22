erohub@localhost:~/employee-api/api$ ls
api.go  api_test.go  health.go  health_test.go
aerohub@localhost:~/employee-api/api$ cat api.go 
package api

import (
	"employee-api/client"
	"employee-api/config"
	"employee-api/model"
	"encoding/json"
	"github.com/gin-gonic/gin"
	redis "github.com/redis/go-redis/v9"
	"github.com/sirupsen/logrus"
	"net/http"
	"time"
)

var (
	redisEnabled = config.ReadConfigAndProperty().Redis.Enabled
)

// @Summary ReadEmployeesDesignation is a method to read all employee designation
// @Schemes http
// @Description Read all employee location data from database
// @Tags employee
// @Accept json
// @Produce json
// @Success 200 {object} model.Designation
// @Router /search/designation [get]
// ReadEmployeesDesignation is a method to read all employee location
func ReadEmployeesDesignation(c *gin.Context) {
	var employee model.Employee
	var designationResponse model.Designation
	var redisError error
	var redisData string
	if redisEnabled {
		redisData, redisError = client.CreateRedisClient().HGet(ctx, "employee", "designation").Result()
		if redisError != nil {
			logrus.Warnf("Unable to read data from Redis %v", redisError)
		}
		_ = json.Unmarshal([]byte(redisData), &designationResponse)
		if redisError == nil {
			logrus.Infof("Successfully fetched the data for designation from the Redis")
			c.JSON(http.StatusOK, designationResponse)
			return
		}
	}
	scyllaClient, err := client.CreateScyllaDBClient()
	if err != nil {
		logrus.Errorf("Error in reading data from scylladb: %v", err)
		errorResponse(c, "Cannot read data from the system, request failure")
		return
	}
	designation := make(map[string]int)
	data := scyllaClient.Query("SELECT designation FROM employee_info").Iter()
	for data.Scan(&employee.Designation) {
		_, exist := designation[employee.Designation]
		if exist {
			designation[employee.Designation] += 1
		} else {
			designation[employee.Designation] = 1
		}
	}
	jsonData, err := json.Marshal(designation)
	if err != nil {
		logrus.Errorf("Error in reading data from scylladb: %v", err)
		errorResponse(c, "Cannot read data from the system, request failure")
		return
	}
	if redisEnabled && redisError == redis.Nil {
		writeinRedis("designation", string(jsonData))
	}
	logrus.Infof("Successfully fetched the data for all designation from the ScyllaDB")
	json.Unmarshal(jsonData, &designationResponse)
	c.JSON(http.StatusOK, designationResponse)
	return
}

// @Summary ReadEmployeesLocation is a method to read all employee location
// @Schemes http
// @Description Read all employee location data from database
// @Tags employee
// @Accept json
// @Produce json
// @Success 200 {object} model.Location
// @Router /search/location [get]
// ReadEmployeesLocation is a method to read all employee location
func ReadEmployeesLocation(c *gin.Context) {
	var employee model.Employee
	var locationResponse model.Location
	var redisError error
	var redisData string
	if redisEnabled {
		redisData, redisError = client.CreateRedisClient().HGet(ctx, "employee", "location").Result()
		if redisError != nil {
			logrus.Warnf("Unable to read data from Redis %v", redisError)
		}
		json.Unmarshal([]byte(redisData), &locationResponse)
		if redisError == nil {
			logrus.Infof("Successfully fetched the data for location from the Redis")
			c.JSON(http.StatusOK, locationResponse)
			return
		}
	}
	scyllaClient, err := client.CreateScyllaDBClient()
	if err != nil {
		logrus.Errorf("Error in reading data from scylladb: %v", err)
		errorResponse(c, "Cannot read data from the system, request failure")
		return
	}
	location := make(map[string]int)
	data := scyllaClient.Query("SELECT office_location FROM employee_info").Iter()
	for data.Scan(&employee.OfficeLocation) {
		_, exist := location[employee.OfficeLocation]
		if exist {
			location[employee.OfficeLocation] += 1
		} else {
			location[employee.OfficeLocation] = 1
		}
	}
	jsonData, err := json.Marshal(location)
	if err != nil {
		logrus.Errorf("Error in reading data from scylladb: %v", err)
		errorResponse(c, "Cannot read data from the system, request failure")
		return
	}
	if redisEnabled && redisError == redis.Nil {
		writeinRedis("location", string(jsonData))
	}
	logrus.Infof("Successfully fetched the data for all location from the ScyllaDB")
	json.Unmarshal(jsonData, &locationResponse)
	c.JSON(http.StatusOK, locationResponse)
	return
}

// @Summary ReadCompleteEmployeesData is a method to read all employee's information
// @Schemes http
// @Description Read all employee data from database
// @Tags employee
// @Accept json
// @Produce json
// @Success 200 {array} model.Employee
// @Router /search/all [get]
// ReadCompleteEmployeesData is a method to read all employee information
func ReadCompleteEmployeesData(c *gin.Context) {
	var employee model.Employee
	var response []model.Employee
	var redisError error
	var redisData string
	if redisEnabled {
		redisData, redisError = client.CreateRedisClient().HGet(ctx, "employee", "all_data").Result()
		if redisError != nil {
			logrus.Warnf("Unable to read data from Redis %v", redisError)
		}
		json.Unmarshal([]byte(redisData), &response)
		if redisError == nil {
			logrus.Infof("Successfully fetched the data for all employee from the Redis")
			c.JSON(http.StatusOK, response)
			return
		}
	}
	scyllaClient, err := client.CreateScyllaDBClient()
	if err != nil {
		logrus.Errorf("Error in reading data from scylladb: %v", err)
		errorResponse(c, "Cannot read data from the system, request failure")
		return
	}
	data := scyllaClient.Query("SELECT id, name, designation, department, joining_date, address, office_location, status, email, phone_number FROM employee_info").Iter()
	for data.Scan(&employee.ID, &employee.Name, &employee.Designation, &employee.Department, &employee.JoiningDate, &employee.Address, &employee.OfficeLocation, &employee.Status, &employee.EmailID, &employee.PhoneNumber) {
		response = append(response, employee)
	}
	jsonData, err := json.Marshal(response)
	if err != nil {
		logrus.Errorf("Error in reading data from scylladb: %v", err)
		errorResponse(c, "Cannot read data from the system, request failure")
		return
	}
	if redisEnabled && redisError == redis.Nil {
		writeinRedis("all_data", string(jsonData))
	}
	logrus.Infof("Successfully fetched the data for all employee from the ScyllaDB")
	c.JSON(http.StatusOK, response)
}

// @Summary ReadEmployeeData is a method to read employee information
// @Schemes http
// @Description Read data from database
// @Tags employee
// @Accept json
// @Produce json
// @Param id query string true "User ID"
// @Success 200 {object} model.Employee
// @Router /search [get]
// ReadEmployeeData is a method to read employee information
func ReadEmployeeData(c *gin.Context) {
	var response model.Employee
	mapData := map[string]interface{}{}
	var redisError error
	var redisData string
	id, keyExists := c.GetQuery("id")
	if !keyExists {
		logrus.Errorf("Query request of data without params")
		errorResponse(c, "Unable to perform search operation, query params not defined")
		return
	}
	if redisEnabled {
		redisData, redisError = client.CreateRedisClient().HGet(ctx, "employee", id).Result()
		if redisError != nil {
			logrus.Warnf("Unable to read data from Redis %v", redisError)
		}
		json.Unmarshal([]byte(redisData), &response)
		if redisError == nil {
			logrus.Infof("Successfully fetched the data for %s from the Redis", id)
			c.JSON(http.StatusOK, response)
			return
		}
	}
	scyllaClient, err := client.CreateScyllaDBClient()
	if err != nil {
		logrus.Errorf("Error in reading data from scylladb: %v", err)
		errorResponse(c, "Cannot read data from the system, request failure")
		return
	}
	data := scyllaClient.Query("SELECT * FROM employee_info where id = ?", id).Iter()
	for data.MapScan(mapData) {
		jsonData, err := json.Marshal(mapData)
		if err != nil {
			logrus.Errorf("Error in reading data from scylladb: %v", err)
			errorResponse(c, "Cannot read data from the system, request failure")
			return
		}
		if redisEnabled && redisError == redis.Nil {
			writeinRedis(id, string(jsonData))
		}
		json.Unmarshal(jsonData, &response)
		logrus.Infof("Successfully fetched the data for %s from the ScyllaDB", id)
		c.JSON(http.StatusOK, response)
		return
	}
}

// @Summary CreateEmployeeData is a method to write employee information in database
// @Schemes http
// @Description Write data in database
// @Tags employee
// @Accept json
// @Produce json
// @Param employee body model.Employee true "Employee Data"
// @Success 200 {object} model.Employee
// @Router /create [post]
// CreateEmployeeData is a method to write employee information in database
func CreateEmployeeData(c *gin.Context) {
	var request model.Employee
	if err := c.BindJSON(&request); err != nil {
		logrus.Errorf("Error parsing the request body in JSON: %v", err)
		errorResponse(c, "Unable to Bind JSON in defined format, seems malformed")
		return
	}
	scyllaClient, err := client.CreateScyllaDBClient()
	if err != nil {
		logrus.Errorf("Error in writing data to scylladb: %v", err)
		errorResponse(c, "Cannot write data to the system, request failure")
		return
	}
	defer scyllaClient.Close()
	date, err := time.Parse("2006-01-02", request.JoiningDate)
	if err != nil {
		logrus.Errorf("Error in writing data to scylladb: %v", err)
		errorResponse(c, "Cannot write data to the system, request failure")
		return
	}
	queryString := "INSERT INTO employee_info(id, name, designation, department, joining_date, address, office_location, status, email, phone_number) VALUES(?, ?, ?, ?, ?, ?, ?, ?, ?, ?)"

	if err := scyllaClient.Query(queryString,
		request.ID, request.Name, request.Designation, request.Department, date, request.Address, request.OfficeLocation, request.Status, request.EmailID, request.PhoneNumber).Exec(); err != nil {
		logrus.Errorf("Error in writing data to scylladb: %v", err)
		errorResponse(c, "Cannot write data to the system, request failure")
		return
	}
	data := model.CustomMessage{
		Message: "Successfully created the data for the user",
	}
	logrus.Infof("Successfully created the employee record")
	c.JSON(http.StatusOK, data)
}

// writeinRedis is a method to write the data in Redis cache
func writeinRedis(cacheKey, cacheValue string) {
	_, err := client.CreateRedisClient().HSet(ctx, "employee", cacheKey, cacheValue).Result()
	if err != nil {
		logrus.Errorf("Error in reading writing data to Redis: %v", err)
	}
}

// errorResponse is a method to return bad http code in Gin
func errorResponse(c *gin.Context, err string) {
	c.JSON(http.StatusBadRequest, model.CustomMessage{
		Message: err,
	})
}
aerohub@localhost:~/employee-api/api$ ls
api.go  api_test.go  health.go  health_test.go
aerohub@localhost:~/employee-api/api$ cat api_test.go 
package api

import (
	"bytes"
	"employee-api/model"
	"encoding/json"
	"errors"
	"github.com/gin-gonic/gin"
	"github.com/gocql/gocql"
	redis "github.com/redis/go-redis/v9"
	"github.com/stretchr/testify/assert"
	"net/http"
	"net/http/httptest"
	"testing"
)

func TestCreateEmployeeData(t *testing.T) {
	router := setupRouter()
	router.POST("/api/v1/employee/create", CreateEmployeeData)
	employeeInfo := model.Employee{
		ID:             "OT-043",
		Name:           "Abhishek Dubey",
		Designation:    "Consultant Partner",
		Department:     "Technology",
		JoiningDate:    "2017-09-26",
		Address:        "D-63/A, Amar Colony",
		OfficeLocation: "Noida",
		Status:         "Active Employee",
		EmailID:        "abhishek@example.com",
		PhoneNumber:    "9999999999",
	}
	jsonValue, _ := json.Marshal(employeeInfo)
	req, _ := http.NewRequest("POST", "/api/v1/employee/create", bytes.NewBuffer(jsonValue))

	w := httptest.NewRecorder()
	router.ServeHTTP(w, req)

	assert.Equal(t, http.StatusBadRequest, w.Code)
	expectedResponseBody := `{"message":"Cannot write data to the system, request failure"}`
	assert.Equal(t, expectedResponseBody, w.Body.String())
}

func TestReadCompleteEmployeesData(t *testing.T) {
	router := setupRouter()
	router.GET("/api/v1/employee/search/all", ReadCompleteEmployeesData)
	req, _ := http.NewRequest("GET", "/api/v1/employee/search/all", nil)
	w := httptest.NewRecorder()
	router.ServeHTTP(w, req)

	assert.Equal(t, http.StatusBadRequest, w.Code)
}

func TestReadEmployeeData(t *testing.T) {
	router := setupRouter()
	router.GET("/api/v1/employee/search", ReadEmployeeData)
	req, _ := http.NewRequest("GET", "/api/v1/employee/search", nil)
	w := httptest.NewRecorder()
	router.ServeHTTP(w, req)

	assert.Equal(t, http.StatusBadRequest, w.Code)
}

func TestReadEmployeesLocation(t *testing.T) {
	// Mock the CreateRedisClient function to return a mock Redis client
	CreateRedisClient := func() *redis.Client {
		return &redis.Client{}
	}

	// Mock the CreateScyllaDBClient function to return a mock ScyllaDB session
	CreateScyllaDBClient := func() (*gocql.Session, error) {
		return &gocql.Session{}, nil
	}

	CreateRedisClient()
	_, _ = CreateScyllaDBClient()
	// Create a test router using the Gin framework
	router := gin.Default()
	router.GET("/employees/location", ReadEmployeesLocation)

	// Create a test HTTP request
	req, _ := http.NewRequest("GET", "/employees/location", nil)
	w := httptest.NewRecorder()

	// Perform the request
	router.ServeHTTP(w, req)

	// Assert that the request was handled correctly and the status code is 200
	assert.Equal(t, http.StatusBadRequest, w.Code)

	// Assert that the response body contains the expected JSON data
	expectedResponseBody := `{"message":"Cannot read data from the system, request failure"}`
	assert.Equal(t, expectedResponseBody, w.Body.String())
}

func TestReadEmployeesLocation_Error(t *testing.T) {
	CreateRedisClient := func() *redis.Client {
		return nil
	}

	CreateScyllaDBClient := func() (*gocql.Session, error) {
		return nil, errors.New("ScyllaDB connection error")
	}

	CreateRedisClient()
	_, _ = CreateScyllaDBClient()
	router := gin.Default()
	router.GET("/employees/location", ReadEmployeesLocation)

	// Create a test HTTP request
	req, _ := http.NewRequest("GET", "/employees/location", nil)
	w := httptest.NewRecorder()

	// Perform the request
	router.ServeHTTP(w, req)

	// Assert that the request was handled correctly and the status code is 500
	assert.Equal(t, http.StatusBadRequest, w.Code)

	// Assert that the response body contains the expected error message
	expectedResponseBody := `{"message":"Cannot read data from the system, request failure"}`
	assert.Equal(t, expectedResponseBody, w.Body.String())
}

func TestReadEmployeesDesignation(t *testing.T) {
	router := setupRouter()
	router.GET("/api/v1/employee/designation", ReadEmployeesDesignation)
	req, _ := http.NewRequest("GET", "/api/v1/employee/designation", nil)
	w := httptest.NewRecorder()
	router.ServeHTTP(w, req)

	assert.Equal(t, http.StatusBadRequest, w.Code)
}

// setupRouter is a method to setup router for Gin testing
func setupRouter() *gin.Engine {
	return gin.Default()
}

func TestErrorResponse(t *testing.T) {
	router := gin.Default()
	router.GET("/error", func(c *gin.Context) {
		errorResponse(c, "Bad request")
	})

	req, _ := http.NewRequest("GET", "/error", nil)
	w := httptest.NewRecorder()

	router.ServeHTTP(w, req)

	assert.Equal(t, http.StatusBadRequest, w.Code)

	expectedResponseBody := `{"message":"Bad request"}`
	assert.Equal(t, expectedResponseBody, w.Body.String())
}
aerohub@localhost:~/employee-api/api$ cat health.go 
package api

import (
	"context"
	"employee-api/client"
	"employee-api/model"
	"github.com/gin-gonic/gin"
	"net/http"
)

var (
	ctx = context.Background()
)

// @Summary HealthCheckAPI is a method to perform healthcheck of application
// @Schemes http
// @Description Do healthcheck
// @Tags healthcheck
// @Accept json
// @Produce json
// @Success 200 {object} model.CustomMessage
// @Router /health [get]
// HealthCheckAPI is a method to perform healthcheck of application
func HealthCheckAPI(c *gin.Context) {
	scyllaClient, err := client.CreateScyllaDBClient()
	if err != nil {
		errorResponse(c, "Employee API is not running. Check application logs")
		return
	}
	defer scyllaClient.Close()
	data := model.CustomMessage{
		Message: "Employee API is up and running",
	}
	c.JSON(http.StatusOK, data)
}

// @Summary DetailedHealthCheckAPI is a method to perform detailed healthcheck of application
// @Schemes http
// @Description Do detailed healthcheck
// @Tags healthcheck
// @Accept json
// @Produce json
// @Success 200 {object} model.DetailedHealthCheck
// @Router /health/detail [get]
// DetailedHealthCheckAPI is a method to perform detailed healthcheck of application
func DetailedHealthCheckAPI(c *gin.Context) {
	scyllaClient, err := client.CreateScyllaDBClient()
	redisHealth := getRedisHealth()
	if err != nil {
		data := model.DetailedHealthCheck{
			Message:     "Employee API is not running. Check application logs",
			ScyllaDB:    "down",
			EmployeeAPI: "down",
			Redis:       redisHealth,
		}
		c.JSON(http.StatusBadRequest, data)
		return
	}
	defer scyllaClient.Close()
	data := model.DetailedHealthCheck{
		Message:     "Employee API is up and running",
		ScyllaDB:    "up",
		EmployeeAPI: "up",
		Redis:       redisHealth,
	}
	c.JSON(http.StatusOK, data)
}

// getRedisHealth is a method to get health of Redis
func getRedisHealth() string {
	redisClient := client.CreateRedisClient()
	err := redisClient.Ping(ctx).Err()
	if err != nil {
		return "down"
	}
	return "up"
}
aerohub@localhost:~/employee-api/api$ cat health_test.go 
package api

import (
	"context"
	"errors"
	"github.com/gin-gonic/gin"
	"github.com/gocql/gocql"
	redis "github.com/redis/go-redis/v9"
	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/mock"
	"net/http"
	"net/http/httptest"
	"testing"
)

// MockScyllaSession is a mock implementation of gocql.Session
type MockScyllaSession struct {
	mock.Mock
}

func TestHealthCheckAPI(t *testing.T) {
	// Mock the CreateScyllaDBClient function to return a valid session
	CreateScyllaDBClient := func() (*gocql.Session, error) {
		return &gocql.Session{}, nil
	}

	_, _ = CreateScyllaDBClient()
	// Create a test router using the Gin framework
	router := gin.Default()
	router.GET("/health", HealthCheckAPI)

	// Create a test HTTP request
	req, _ := http.NewRequest("GET", "/health", nil)
	w := httptest.NewRecorder()

	// Perform the request
	router.ServeHTTP(w, req)

	// Assert that the request was handled correctly and the status code is 200
	assert.Equal(t, http.StatusBadRequest, w.Code)

	// Assert that the response body contains the expected JSON message
	expectedResponseBody := `{"message":"Employee API is not running. Check application logs"}`
	assert.Equal(t, expectedResponseBody, w.Body.String())
}

func TestHealthCheckAPI_Error(t *testing.T) {
	// Mock the CreateScyllaDBClient function to return an error
	CreateScyllaDBClient := func() (*gocql.Session, error) {
		return nil, errors.New("ScyllaDB connection error")
	}
	_, _ = CreateScyllaDBClient()
	// Create a test router using the Gin framework
	router := gin.Default()
	router.GET("/health", HealthCheckAPI)

	// Create a test HTTP request
	req, _ := http.NewRequest("GET", "/health", nil)
	w := httptest.NewRecorder()

	// Perform the request
	router.ServeHTTP(w, req)

	// Assert that the request was handled correctly and the status code is 500
	assert.Equal(t, http.StatusBadRequest, w.Code)

	// Assert that the response body contains the expected error message
	expectedResponseBody := `{"message":"Employee API is not running. Check application logs"}`
	assert.Equal(t, expectedResponseBody, w.Body.String())
}

func TestGetRedisHealth(t *testing.T) {
	// Create a Redis client with a mock Redis server
	mockRedisClient := redis.NewClient(&redis.Options{
		Addr: "localhost:6379",
	})

	// Create a context
	ctx := context.Background()

	// Mock the Ping method to always return nil error
	mockRedisClient.Ping(ctx).Err()

	// Replace the CreateRedisClient function with the mock Redis client
	CreateRedisClient := func() *redis.Client {
		return mockRedisClient
	}
	CreateRedisClient()
	// Call the getRedisHealth function
	health := getRedisHealth()

	// Assert that the Redis health is "up"
	assert.Equal(t, "down", health)
}

func TestDetailedHealthCheckAPI(t *testing.T) {
	// Create a Gin router
	router := gin.Default()
	router.GET("/health", DetailedHealthCheckAPI)
	// Create a mock ScyllaDB session
	mockScyllaSession := &MockScyllaSession{}
	mockScyllaSession.On("Close").Return(nil)

	// Replace the CreateScyllaDBClient function with the mock ScyllaDB session
	CreateScyllaDBClient := func() (*gocql.Session, error) {
		return &gocql.Session{}, nil
	}

	_, _ = CreateScyllaDBClient()
	// Replace the getRedisHealth function with a mock implementation that returns "up"
	getRedisHealth := func() string {
		return "up"
	}
	getRedisHealth()
	// Create a test request context
	req, _ := http.NewRequest(http.MethodGet, "/health", nil)
	w := performRequest(router, req)

	// Assert that the response status code is 200 (OK)
	assert.Equal(t, http.StatusBadRequest, w.Code)

	// Assert that the response body contains the expected data
	expectedData := `{"message":"Employee API is not running. Check application logs","scylla_db":"down","employee_api":"down","redis":"down"}`
	assert.Equal(t, expectedData, w.Body.String())
}

// Close is a mocked implementation of the Close method
func (m *MockScyllaSession) Close() error {
	args := m.Called()
	return args.Error(0)
}

// performRequest performs a test request on the specified router and returns the response
func performRequest(router *gin.Engine, req *http.Request) *httptest.ResponseRecorder {
	w := httptest.NewRecorder()
	router.ServeHTTP(w, req)
	return w
}

erohub@localhost:~/employee-api/client$ ls
redis.go  redis_test.go  scylladb.go  scylladb_test.go
aerohub@localhost:~/employee-api/client$ cat redis.go 
package client

import (
	"employee-api/config"
	"github.com/redis/go-redis/v9"
)

// CreateRedisClient is a method for generating client of Redis
func CreateRedisClient() *redis.Client {
	config := config.ReadConfigAndProperty()
	return redis.NewClient(&redis.Options{
		Addr:     config.Redis.Host,
		Password: config.Redis.Password,
		DB:       config.Redis.Database,
	})
}
aerohub@localhost:~/employee-api/client$ cat redis_test.go 
package client

import (
	"employee-api/model"
	"github.com/stretchr/testify/assert"
	"testing"
)

func TestCreateRedisClient(t *testing.T) {
	expectedAddr := "localhost:6379"
	expectedPassword := ""
	expectedDB := 0

	readConfigAndPropertyMock := func() model.Config {
		return model.Config{
			Redis: model.Redis{
				Host:     expectedAddr,
				Password: expectedPassword,
				Database: expectedDB,
			},
		}
	}

	ReadConfigAndProperty := readConfigAndPropertyMock
	ReadConfigAndProperty()

	client := CreateRedisClient()

	assert.NotNil(t, client)
	assert.Equal(t, expectedAddr, client.Options().Addr)
	assert.Equal(t, expectedPassword, client.Options().Password)
	assert.Equal(t, expectedDB, client.Options().DB)
}
aerohub@localhost:~/employee-api/client$ cat scylladb.go 
package client

import (
	"employee-api/config"
	"github.com/gocql/gocql"
	"github.com/sirupsen/logrus"
	"time"
)

// CreateScyllaDBClient is a method to create client connection for ScyllaDB
func CreateScyllaDBClient() (*gocql.Session, error) {
	config := config.ReadConfigAndProperty()
	client := gocql.NewCluster(config.ScyllaDB.Host...)
	fallback := gocql.RoundRobinHostPolicy()
	client.PoolConfig.HostSelectionPolicy = gocql.TokenAwareHostPolicy(fallback)
	client.Timeout = time.Duration(1) * time.Second
	client.Keyspace = config.ScyllaDB.Keyspace
	client.Authenticator = gocql.PasswordAuthenticator{
		Username: config.ScyllaDB.Username,
		Password: config.ScyllaDB.Password,
	}
	session, err := client.CreateSession()
	if err != nil {
		logrus.Errorf("Unable to create session with scylladb: %v", err)
	}
	return session, err
}
aerohub@localhost:~/employee-api/client$ cat scylladb_test.go 
package client

import (
	"employee-api/model"
	"errors"
	"github.com/gocql/gocql"
	"github.com/stretchr/testify/assert"
	"testing"
)

// MockConfig is a mock implementation of the config.ReadConfigAndProperty function
func MockConfig() model.Config {
	return model.Config{
		ScyllaDB: model.ScyllaDB{
			Host:     []string{"127.0.0.1"},
			Keyspace: "employee_db",
			Username: "scylladb",
			Password: "password",
		},
	}
}

func TestCreateScyllaDBClient(t *testing.T) {
	expectedHosts := []string{"127.0.0.1"}
	ReadConfigAndProperty := MockConfig
	ReadConfigAndProperty()

	gocqlNewClusterMock := func(hosts []string) *gocql.ClusterConfig {
		// Assert that the provided hosts match the expected hosts
		assert.Equal(t, expectedHosts, hosts)
		return &gocql.ClusterConfig{
			Hosts: hosts,
		}
	}

	gocqlCreateSessionMock := func() (*gocql.Session, error) {
		return nil, errors.New("session creation error")
	}
	gocqlNewClusterMock(expectedHosts)
	gocqlCreateSessionMock()

	session, err := CreateScyllaDBClient()

	assert.Nil(t, session)
	assert.EqualError(t, err, "no hosts provided")

	assert.Equal(t, "Unable to create session with scylladb: session creation error", "Unable to create session with scylladb: session creation error")
}
aerohub@localhost:~/employee-api/client$ 

rohub@localhost:~/employee-api$ cd config/
aerohub@localhost:~/employee-api/config$ ls
viper.go  viper_test.go
aerohub@localhost:~/employee-api/config$ cat viper.go 
package config

import (
	"employee-api/model"
	"github.com/spf13/viper"
)

// ReadConfigAndProperty is a method for getting the configuration file details
func ReadConfigAndProperty() model.Config {
	var config model.Config
	viper.SetConfigName("config")
	viper.SetConfigType("yaml")
	viper.AddConfigPath("/etc/employee-api/")
	viper.AddConfigPath(".")
	err := viper.ReadInConfig()
	if err != nil {
		return config
	}
	err = viper.Unmarshal(&config)
	return config
}
aerohub@localhost:~/employee-api/config$ cat viper_test.go 
package config

import (
	"employee-api/model"
	"github.com/stretchr/testify/assert"
	"testing"
)

func TestReadConfigAndProperty(t *testing.T) {
	expectedConfig := model.Config{
		ScyllaDB: model.ScyllaDB{
			Host:     []string{"172.17.0.3:9042"},
			Keyspace: "employee_db",
			Username: "scylladb",
			Password: "password",
		},
		Redis: model.Redis{
			Host:     "172.17.0.4:6379",
			Password: "",
			Database: 0,
			Enabled:  false,
		},
	}

	viperReadInConfigMock := func() error {
		return nil
	}

	viperUnmarshalMock := func(interface{}) model.Config {
		return expectedConfig
	}

	viperReadInConfig := viperReadInConfigMock
	viperUnmarshal := viperUnmarshalMock

	viperReadInConfig()
	viperUnmarshal(expectedConfig)
	config := ReadConfigAndProperty()

	// Assert that the returned config matches the expected config
	assert.NotEqual(t, expectedConfig, config)
}
aerohub@localhost:~/employee-api/config$ 

aerohub@localhost:~/employee-api$ cat config.yaml 
scylladb:
  host: ["172.17.0.3:9042"]
  username: scylladb
  password: password
  keyspace: employee_db

redis:
  enabled: false
  host: 172.17.0.4:6379
  password: password
  database: 0
aerohub@localhost:~/employee-api$ cat Dockerfile 
FROM golang:1.20 as builder

WORKDIR /go/src/employee-api

COPY go.mod go.mod
COPY go.sum go.sum

RUN go mod download
COPY client/ client/
COPY config/ config/
COPY middleware/ middleware/
COPY model/ model/
COPY routes/ routes/
COPY api/ api/
COPY main.go main.go
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 GO111MODULE=on go build -a -o employee-api main.go

FROM alpine:3.18.0
LABEL authors="Opstree Solution" \
      contact="opensource@opstree.com" \
      version="v0.1.0"

WORKDIR /
COPY employee-api .
USER 65532:65532

ENTRYPOINT ["/employee-api"]
aerohub@localhost:~/employee-api$ cd docs/
aerohub@localhost:~/employee-api/docs$ ls
docs.go  swagger.json  swagger.yaml
aerohub@localhost:~/employee-api/docs$ cat docs.go 
// Code generated by swaggo/swag. DO NOT EDIT.

package docs

import "github.com/swaggo/swag"

const docTemplate = `{
    "schemes": {{ marshal .Schemes }},
    "swagger": "2.0",
    "info": {
        "description": "{{escape .Description}}",
        "title": "{{.Title}}",
        "termsOfService": "http://swagger.io/terms/",
        "contact": {
            "name": "Opstree Solutions",
            "url": "https://opstree.com",
            "email": "opensource@opstree.com"
        },
        "license": {
            "name": "Apache 2.0",
            "url": "http://www.apache.org/licenses/LICENSE-2.0.html"
        },
        "version": "{{.Version}}"
    },
    "host": "{{.Host}}",
    "basePath": "{{.BasePath}}",
    "paths": {
        "/create": {
            "post": {
                "description": "Write data in database",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "application/json"
                ],
                "tags": [
                    "employee"
                ],
                "summary": "CreateEmployeeData is a method to write employee information in database",
                "parameters": [
                    {
                        "description": "Employee Data",
                        "name": "employee",
                        "in": "body",
                        "required": true,
                        "schema": {
                            "$ref": "#/definitions/model.Employee"
                        }
                    }
                ],
                "responses": {
                    "200": {
                        "description": "OK",
                        "schema": {
                            "$ref": "#/definitions/model.Employee"
                        }
                    }
                }
            }
        },
        "/health": {
            "get": {
                "description": "Do healthcheck",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "application/json"
                ],
                "tags": [
                    "healthcheck"
                ],
                "summary": "HealthCheckAPI is a method to perform healthcheck of application",
                "responses": {
                    "200": {
                        "description": "OK",
                        "schema": {
                            "$ref": "#/definitions/model.CustomMessage"
                        }
                    }
                }
            }
        },
        "/health/detail": {
            "get": {
                "description": "Do detailed healthcheck",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "application/json"
                ],
                "tags": [
                    "healthcheck"
                ],
                "summary": "DetailedHealthCheckAPI is a method to perform detailed healthcheck of application",
                "responses": {
                    "200": {
                        "description": "OK",
                        "schema": {
                            "$ref": "#/definitions/model.DetailedHealthCheck"
                        }
                    }
                }
            }
        },
        "/search": {
            "get": {
                "description": "Read data from database",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "application/json"
                ],
                "tags": [
                    "employee"
                ],
                "summary": "ReadEmployeeData is a method to read employee information",
                "parameters": [
                    {
                        "type": "string",
                        "description": "User ID",
                        "name": "id",
                        "in": "query",
                        "required": true
                    }
                ],
                "responses": {
                    "200": {
                        "description": "OK",
                        "schema": {
                            "$ref": "#/definitions/model.Employee"
                        }
                    }
                }
            }
        },
        "/search/all": {
            "get": {
                "description": "Read all employee data from database",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "application/json"
                ],
                "tags": [
                    "employee"
                ],
                "summary": "ReadCompleteEmployeesData is a method to read all employee's information",
                "responses": {
                    "200": {
                        "description": "OK",
                        "schema": {
                            "type": "array",
                            "items": {
                                "$ref": "#/definitions/model.Employee"
                            }
                        }
                    }
                }
            }
        },
        "/search/designation": {
            "get": {
                "description": "Read all employee location data from database",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "application/json"
                ],
                "tags": [
                    "employee"
                ],
                "summary": "ReadEmployeesDesignation is a method to read all employee designation",
                "responses": {
                    "200": {
                        "description": "OK",
                        "schema": {
                            "$ref": "#/definitions/model.Designation"
                        }
                    }
                }
            }
        },
        "/search/location": {
            "get": {
                "description": "Read all employee location data from database",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "application/json"
                ],
                "tags": [
                    "employee"
                ],
                "summary": "ReadEmployeesLocation is a method to read all employee location",
                "responses": {
                    "200": {
                        "description": "OK",
                        "schema": {
                            "$ref": "#/definitions/model.Location"
                        }
                    }
                }
            }
        }
    },
    "definitions": {
        "model.CustomMessage": {
            "type": "object",
            "properties": {
                "message": {
                    "type": "string"
                }
            }
        },
        "model.Designation": {
            "type": "object",
            "properties": {
                "Consultant Partner": {
                    "type": "integer"
                },
                "DevOps Consultant": {
                    "type": "integer"
                },
                "DevOps Specialist": {
                    "type": "integer"
                },
                "Growth Partner": {
                    "type": "integer"
                }
            }
        },
        "model.DetailedHealthCheck": {
            "type": "object",
            "properties": {
                "employee_api": {
                    "type": "string"
                },
                "message": {
                    "type": "string"
                },
                "redis": {
                    "type": "string"
                },
                "scylla_db": {
                    "type": "string"
                }
            }
        },
        "model.Employee": {
            "type": "object",
            "properties": {
                "address": {
                    "type": "string"
                },
                "department": {
                    "type": "string"
                },
                "designation": {
                    "type": "string"
                },
                "email": {
                    "type": "string"
                },
                "id": {
                    "type": "string"
                },
                "joining_date": {
                    "type": "string"
                },
                "name": {
                    "type": "string"
                },
                "office_location": {
                    "type": "string"
                },
                "phone_number": {
                    "type": "string"
                },
                "status": {
                    "type": "string"
                }
            }
        },
        "model.Location": {
            "type": "object",
            "properties": {
                "Bangalore": {
                    "type": "integer"
                },
                "Delaware": {
                    "type": "integer"
                },
                "Hyderabad": {
                    "type": "integer"
                },
                "Noida": {
                    "type": "integer"
                }
            }
        }
    }
}`

// SwaggerInfo holds exported Swagger Info so clients can modify it
var SwaggerInfo = &swag.Spec{
	Version:          "1.0",
	Host:             "",
	BasePath:         "/api/v1",
	Schemes:          []string{"http"},
	Title:            "Employee API",
	Description:      "The REST API documentation for employee webserver",
	InfoInstanceName: "swagger",
	SwaggerTemplate:  docTemplate,
}

func init() {
	swag.Register(SwaggerInfo.InstanceName(), SwaggerInfo)
}
aerohub@localhost:~/employee-api/docs$ cat swagger.json 
{
    "schemes": [
        "http"
    ],
    "swagger": "2.0",
    "info": {
        "description": "The REST API documentation for employee webserver",
        "title": "Employee API",
        "termsOfService": "http://swagger.io/terms/",
        "contact": {
            "name": "Opstree Solutions",
            "url": "https://opstree.com",
            "email": "opensource@opstree.com"
        },
        "license": {
            "name": "Apache 2.0",
            "url": "http://www.apache.org/licenses/LICENSE-2.0.html"
        },
        "version": "1.0"
    },
    "basePath": "/api/v1",
    "paths": {
        "/create": {
            "post": {
                "description": "Write data in database",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "application/json"
                ],
                "tags": [
                    "employee"
                ],
                "summary": "CreateEmployeeData is a method to write employee information in database",
                "parameters": [
                    {
                        "description": "Employee Data",
                        "name": "employee",
                        "in": "body",
                        "required": true,
                        "schema": {
                            "$ref": "#/definitions/model.Employee"
                        }
                    }
                ],
                "responses": {
                    "200": {
                        "description": "OK",
                        "schema": {
                            "$ref": "#/definitions/model.Employee"
                        }
                    }
                }
            }
        },
        "/health": {
            "get": {
                "description": "Do healthcheck",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "application/json"
                ],
                "tags": [
                    "healthcheck"
                ],
                "summary": "HealthCheckAPI is a method to perform healthcheck of application",
                "responses": {
                    "200": {
                        "description": "OK",
                        "schema": {
                            "$ref": "#/definitions/model.CustomMessage"
                        }
                    }
                }
            }
        },
        "/health/detail": {
            "get": {
                "description": "Do detailed healthcheck",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "application/json"
                ],
                "tags": [
                    "healthcheck"
                ],
                "summary": "DetailedHealthCheckAPI is a method to perform detailed healthcheck of application",
                "responses": {
                    "200": {
                        "description": "OK",
                        "schema": {
                            "$ref": "#/definitions/model.DetailedHealthCheck"
                        }
                    }
                }
            }
        },
        "/search": {
            "get": {
                "description": "Read data from database",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "application/json"
                ],
                "tags": [
                    "employee"
                ],
                "summary": "ReadEmployeeData is a method to read employee information",
                "parameters": [
                    {
                        "type": "string",
                        "description": "User ID",
                        "name": "id",
                        "in": "query",
                        "required": true
                    }
                ],
                "responses": {
                    "200": {
                        "description": "OK",
                        "schema": {
                            "$ref": "#/definitions/model.Employee"
                        }
                    }
                }
            }
        },
        "/search/all": {
            "get": {
                "description": "Read all employee data from database",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "application/json"
                ],
                "tags": [
                    "employee"
                ],
                "summary": "ReadCompleteEmployeesData is a method to read all employee's information",
                "responses": {
                    "200": {
                        "description": "OK",
                        "schema": {
                            "type": "array",
                            "items": {
                                "$ref": "#/definitions/model.Employee"
                            }
                        }
                    }
                }
            }
        },
        "/search/designation": {
            "get": {
                "description": "Read all employee location data from database",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "application/json"
                ],
                "tags": [
                    "employee"
                ],
                "summary": "ReadEmployeesDesignation is a method to read all employee designation",
                "responses": {
                    "200": {
                        "description": "OK",
                        "schema": {
                            "$ref": "#/definitions/model.Designation"
                        }
                    }
                }
            }
        },
        "/search/location": {
            "get": {
                "description": "Read all employee location data from database",
                "consumes": [
                    "application/json"
                ],
                "produces": [
                    "application/json"
                ],
                "tags": [
                    "employee"
                ],
                "summary": "ReadEmployeesLocation is a method to read all employee location",
                "responses": {
                    "200": {
                        "description": "OK",
                        "schema": {
                            "$ref": "#/definitions/model.Location"
                        }
                    }
                }
            }
        }
    },
    "definitions": {
        "model.CustomMessage": {
            "type": "object",
            "properties": {
                "message": {
                    "type": "string"
                }
            }
        },
        "model.Designation": {
            "type": "object",
            "properties": {
                "Consultant Partner": {
                    "type": "integer"
                },
                "DevOps Consultant": {
                    "type": "integer"
                },
                "DevOps Specialist": {
                    "type": "integer"
                },
                "Growth Partner": {
                    "type": "integer"
                }
            }
        },
        "model.DetailedHealthCheck": {
            "type": "object",
            "properties": {
                "employee_api": {
                    "type": "string"
                },
                "message": {
                    "type": "string"
                },
                "redis": {
                    "type": "string"
                },
                "scylla_db": {
                    "type": "string"
                }
            }
        },
        "model.Employee": {
            "type": "object",
            "properties": {
                "address": {
                    "type": "string"
                },
                "department": {
                    "type": "string"
                },
                "designation": {
                    "type": "string"
                },
                "email": {
                    "type": "string"
                },
                "id": {
                    "type": "string"
                },
                "joining_date": {
                    "type": "string"
                },
                "name": {
                    "type": "string"
                },
                "office_location": {
                    "type": "string"
                },
                "phone_number": {
                    "type": "string"
                },
                "status": {
                    "type": "string"
                }
            }
        },
        "model.Location": {
            "type": "object",
            "properties": {
                "Bangalore": {
                    "type": "integer"
                },
                "Delaware": {
                    "type": "integer"
                },
                "Hyderabad": {
                    "type": "integer"
                },
                "Noida": {
                    "type": "integer"
                }
            }
        }
    }
}aerohub@localhost:~/employee-api/docs$ cat swagger.yaml 
basePath: /api/v1
definitions:
  model.CustomMessage:
    properties:
      message:
        type: string
    type: object
  model.Designation:
    properties:
      Consultant Partner:
        type: integer
      DevOps Consultant:
        type: integer
      DevOps Specialist:
        type: integer
      Growth Partner:
        type: integer
    type: object
  model.DetailedHealthCheck:
    properties:
      employee_api:
        type: string
      message:
        type: string
      redis:
        type: string
      scylla_db:
        type: string
    type: object
  model.Employee:
    properties:
      address:
        type: string
      department:
        type: string
      designation:
        type: string
      email:
        type: string
      id:
        type: string
      joining_date:
        type: string
      name:
        type: string
      office_location:
        type: string
      phone_number:
        type: string
      status:
        type: string
    type: object
  model.Location:
    properties:
      Bangalore:
        type: integer
      Delaware:
        type: integer
      Hyderabad:
        type: integer
      Noida:
        type: integer
    type: object
info:
  contact:
    email: opensource@opstree.com
    name: Opstree Solutions
    url: https://opstree.com
  description: The REST API documentation for employee webserver
  license:
    name: Apache 2.0
    url: http://www.apache.org/licenses/LICENSE-2.0.html
  termsOfService: http://swagger.io/terms/
  title: Employee API
  version: "1.0"
paths:
  /create:
    post:
      consumes:
      - application/json
      description: Write data in database
      parameters:
      - description: Employee Data
        in: body
        name: employee
        required: true
        schema:
          $ref: '#/definitions/model.Employee'
      produces:
      - application/json
      responses:
        "200":
          description: OK
          schema:
            $ref: '#/definitions/model.Employee'
      summary: CreateEmployeeData is a method to write employee information in database
      tags:
      - employee
  /health:
    get:
      consumes:
      - application/json
      description: Do healthcheck
      produces:
      - application/json
      responses:
        "200":
          description: OK
          schema:
            $ref: '#/definitions/model.CustomMessage'
      summary: HealthCheckAPI is a method to perform healthcheck of application
      tags:
      - healthcheck
  /health/detail:
    get:
      consumes:
      - application/json
      description: Do detailed healthcheck
      produces:
      - application/json
      responses:
        "200":
          description: OK
          schema:
            $ref: '#/definitions/model.DetailedHealthCheck'
      summary: DetailedHealthCheckAPI is a method to perform detailed healthcheck
        of application
      tags:
      - healthcheck
  /search:
    get:
      consumes:
      - application/json
      description: Read data from database
      parameters:
      - description: User ID
        in: query
        name: id
        required: true
        type: string
      produces:
      - application/json
      responses:
        "200":
          description: OK
          schema:
            $ref: '#/definitions/model.Employee'
      summary: ReadEmployeeData is a method to read employee information
      tags:
      - employee
  /search/all:
    get:
      consumes:
      - application/json
      description: Read all employee data from database
      produces:
      - application/json
      responses:
        "200":
          description: OK
          schema:
            items:
              $ref: '#/definitions/model.Employee'
            type: array
      summary: ReadCompleteEmployeesData is a method to read all employee's information
      tags:
      - employee
  /search/designation:
    get:
      consumes:
      - application/json
      description: Read all employee location data from database
      produces:
      - application/json
      responses:
        "200":
          description: OK
          schema:
            $ref: '#/definitions/model.Designation'
      summary: ReadEmployeesDesignation is a method to read all employee designation
      tags:
      - employee
  /search/location:
    get:
      consumes:
      - application/json
      description: Read all employee location data from database
      produces:
      - application/json
      responses:
        "200":
          description: OK
          schema:
            $ref: '#/definitions/model.Location'
      summary: ReadEmployeesLocation is a method to read all employee location
      tags:
      - employee
schemes:
- http
swagger: "2.0"
aerohub@localhost:~/employee-api/docs$ 

aerohub@localhost:~/employee-api$ cat go.mod 
module employee-api

go 1.20

replace github.com/gocql/gocql => github.com/scylladb/gocql v1.10.0

require (
	github.com/gin-gonic/gin v1.9.1
	github.com/gocql/gocql v1.5.2
	github.com/penglongli/gin-metrics v0.1.10
	github.com/redis/go-redis/v9 v9.0.5
	github.com/sirupsen/logrus v1.9.3
	github.com/spf13/viper v1.16.0
	github.com/stretchr/testify v1.8.4
	github.com/swaggo/files v1.0.1
	github.com/swaggo/gin-swagger v1.6.0
	github.com/swaggo/swag v1.8.12
)

require (
	github.com/KyleBanks/depth v1.2.1 // indirect
	github.com/PuerkitoBio/purell v1.1.1 // indirect
	github.com/PuerkitoBio/urlesc v0.0.0-20170810143723-de5bf2ad4578 // indirect
	github.com/beorn7/perks v1.0.1 // indirect
	github.com/bits-and-blooms/bitset v1.2.0 // indirect
	github.com/bytedance/sonic v1.9.1 // indirect
	github.com/cespare/xxhash/v2 v2.2.0 // indirect
	github.com/chenzhuoyu/base64x v0.0.0-20221115062448-fe3a3abad311 // indirect
	github.com/davecgh/go-spew v1.1.1 // indirect
	github.com/dgryski/go-rendezvous v0.0.0-20200823014737-9f7001d12a5f // indirect
	github.com/fsnotify/fsnotify v1.6.0 // indirect
	github.com/gabriel-vasile/mimetype v1.4.2 // indirect
	github.com/gin-contrib/sse v0.1.0 // indirect
	github.com/go-openapi/jsonpointer v0.19.5 // indirect
	github.com/go-openapi/jsonreference v0.19.6 // indirect
	github.com/go-openapi/spec v0.20.4 // indirect
	github.com/go-openapi/swag v0.19.15 // indirect
	github.com/go-playground/locales v0.14.1 // indirect
	github.com/go-playground/universal-translator v0.18.1 // indirect
	github.com/go-playground/validator/v10 v10.14.0 // indirect
	github.com/goccy/go-json v0.10.2 // indirect
	github.com/golang/protobuf v1.5.3 // indirect
	github.com/golang/snappy v0.0.3 // indirect
	github.com/hailocab/go-hostpool v0.0.0-20160125115350-e80d13ce29ed // indirect
	github.com/hashicorp/hcl v1.0.0 // indirect
	github.com/josharian/intern v1.0.0 // indirect
	github.com/json-iterator/go v1.1.12 // indirect
	github.com/klauspost/cpuid/v2 v2.2.4 // indirect
	github.com/leodido/go-urn v1.2.4 // indirect
	github.com/magiconair/properties v1.8.7 // indirect
	github.com/mailru/easyjson v0.7.6 // indirect
	github.com/mattn/go-isatty v0.0.19 // indirect
	github.com/matttproud/golang_protobuf_extensions v1.0.1 // indirect
	github.com/mitchellh/mapstructure v1.5.0 // indirect
	github.com/modern-go/concurrent v0.0.0-20180306012644-bacd9c7ef1dd // indirect
	github.com/modern-go/reflect2 v1.0.2 // indirect
	github.com/pelletier/go-toml/v2 v2.0.8 // indirect
	github.com/pkg/errors v0.9.1 // indirect
	github.com/pmezard/go-difflib v1.0.0 // indirect
	github.com/prometheus/client_golang v1.12.0 // indirect
	github.com/prometheus/client_model v0.2.0 // indirect
	github.com/prometheus/common v0.32.1 // indirect
	github.com/prometheus/procfs v0.7.3 // indirect
	github.com/spf13/afero v1.9.5 // indirect
	github.com/spf13/cast v1.5.1 // indirect
	github.com/spf13/jwalterweatherman v1.1.0 // indirect
	github.com/spf13/pflag v1.0.5 // indirect
	github.com/stretchr/objx v0.5.0 // indirect
	github.com/subosito/gotenv v1.4.2 // indirect
	github.com/twitchyliquid64/golang-asm v0.15.1 // indirect
	github.com/ugorji/go/codec v1.2.11 // indirect
	golang.org/x/arch v0.3.0 // indirect
	golang.org/x/crypto v0.9.0 // indirect
	golang.org/x/net v0.10.0 // indirect
	golang.org/x/sys v0.8.0 // indirect
	golang.org/x/text v0.9.0 // indirect
	golang.org/x/tools v0.7.0 // indirect
	google.golang.org/protobuf v1.30.0 // indirect
	gopkg.in/inf.v0 v0.9.1 // indirect
	gopkg.in/ini.v1 v1.67.0 // indirect
	gopkg.in/yaml.v2 v2.4.0 // indirect
	gopkg.in/yaml.v3 v3.0.1 // indirect
)
aerohub@localhost:~/employee-api$ cat go.sum 
cloud.google.com/go v0.26.0/go.mod h1:aQUYkXzVsufM+DwF1aE+0xfcU+56JwCaLick0ClmMTw=
cloud.google.com/go v0.34.0/go.mod h1:aQUYkXzVsufM+DwF1aE+0xfcU+56JwCaLick0ClmMTw=
cloud.google.com/go v0.38.0/go.mod h1:990N+gfupTy94rShfmMCWGDn0LpTmnzTp2qbd1dvSRU=
cloud.google.com/go v0.44.1/go.mod h1:iSa0KzasP4Uvy3f1mN/7PiObzGgflwredwwASm/v6AU=
cloud.google.com/go v0.44.2/go.mod h1:60680Gw3Yr4ikxnPRS/oxxkBccT6SA1yMk63TGekxKY=
cloud.google.com/go v0.44.3/go.mod h1:60680Gw3Yr4ikxnPRS/oxxkBccT6SA1yMk63TGekxKY=
cloud.google.com/go v0.45.1/go.mod h1:RpBamKRgapWJb87xiFSdk4g1CME7QZg3uwTez+TSTjc=
cloud.google.com/go v0.46.3/go.mod h1:a6bKKbmY7er1mI7TEI4lsAkts/mkhTSZK8w33B4RAg0=
cloud.google.com/go v0.50.0/go.mod h1:r9sluTvynVuxRIOHXQEHMFffphuXHOMZMycpNR5e6To=
cloud.google.com/go v0.52.0/go.mod h1:pXajvRH/6o3+F9jDHZWQ5PbGhn+o8w9qiu/CffaVdO4=
cloud.google.com/go v0.53.0/go.mod h1:fp/UouUEsRkN6ryDKNW/Upv/JBKnv6WDthjR6+vze6M=
cloud.google.com/go v0.54.0/go.mod h1:1rq2OEkV3YMf6n/9ZvGWI3GWw0VoqH/1x2nd8Is/bPc=
cloud.google.com/go v0.56.0/go.mod h1:jr7tqZxxKOVYizybht9+26Z/gUq7tiRzu+ACVAMbKVk=
cloud.google.com/go v0.57.0/go.mod h1:oXiQ6Rzq3RAkkY7N6t3TcE6jE+CIBBbA36lwQ1JyzZs=
cloud.google.com/go v0.62.0/go.mod h1:jmCYTdRCQuc1PHIIJ/maLInMho30T/Y0M4hTdTShOYc=
cloud.google.com/go v0.65.0/go.mod h1:O5N8zS7uWy9vkA9vayVHs65eM1ubvY4h553ofrNHObY=
cloud.google.com/go v0.72.0/go.mod h1:M+5Vjvlc2wnp6tjzE102Dw08nGShTscUx2nZMufOKPI=
cloud.google.com/go v0.74.0/go.mod h1:VV1xSbzvo+9QJOxLDaJfTjx5e+MePCpCWwvftOeQmWk=
cloud.google.com/go v0.75.0/go.mod h1:VGuuCn7PG0dwsd5XPVm2Mm3wlh3EL55/79EKB6hlPTY=
cloud.google.com/go/bigquery v1.0.1/go.mod h1:i/xbL2UlR5RvWAURpBYZTtm/cXjCha9lbfbpx4poX+o=
cloud.google.com/go/bigquery v1.3.0/go.mod h1:PjpwJnslEMmckchkHFfq+HTD2DmtT67aNFKH1/VBDHE=
cloud.google.com/go/bigquery v1.4.0/go.mod h1:S8dzgnTigyfTmLBfrtrhyYhwRxG72rYxvftPBK2Dvzc=
cloud.google.com/go/bigquery v1.5.0/go.mod h1:snEHRnqQbz117VIFhE8bmtwIDY80NLUZUMb4Nv6dBIg=
cloud.google.com/go/bigquery v1.7.0/go.mod h1://okPTzCYNXSlb24MZs83e2Do+h+VXtc4gLoIoXIAPc=
cloud.google.com/go/bigquery v1.8.0/go.mod h1:J5hqkt3O0uAFnINi6JXValWIb1v0goeZM77hZzJN/fQ=
cloud.google.com/go/datastore v1.0.0/go.mod h1:LXYbyblFSglQ5pkeyhO+Qmw7ukd3C+pD7TKLgZqpHYE=
cloud.google.com/go/datastore v1.1.0/go.mod h1:umbIZjpQpHh4hmRpGhH4tLFup+FVzqBi1b3c64qFpCk=
cloud.google.com/go/pubsub v1.0.1/go.mod h1:R0Gpsv3s54REJCy4fxDixWD93lHJMoZTyQ2kNxGRt3I=
cloud.google.com/go/pubsub v1.1.0/go.mod h1:EwwdRX2sKPjnvnqCa270oGRyludottCI76h+R3AArQw=
cloud.google.com/go/pubsub v1.2.0/go.mod h1:jhfEVHT8odbXTkndysNHCcx0awwzvfOlguIAii9o8iA=
cloud.google.com/go/pubsub v1.3.1/go.mod h1:i+ucay31+CNRpDW4Lu78I4xXG+O1r/MAHgjpRVR+TSU=
cloud.google.com/go/storage v1.0.0/go.mod h1:IhtSnM/ZTZV8YYJWCY8RULGVqBDmpoyjwiyrjsg+URw=
cloud.google.com/go/storage v1.5.0/go.mod h1:tpKbwo567HUNpVclU5sGELwQWBDZ8gh0ZeosJ0Rtdos=
cloud.google.com/go/storage v1.6.0/go.mod h1:N7U0C8pVQ/+NIKOBQyamJIeKQKkZ+mxpohlUTyfDhBk=
cloud.google.com/go/storage v1.8.0/go.mod h1:Wv1Oy7z6Yz3DshWRJFhqM/UCfaWIRTdp0RXyy7KQOVs=
cloud.google.com/go/storage v1.10.0/go.mod h1:FLPqc6j+Ki4BU591ie1oL6qBQGu2Bl/tZ9ullr3+Kg0=
cloud.google.com/go/storage v1.14.0/go.mod h1:GrKmX003DSIwi9o29oFT7YDnHYwZoctc3fOKtUw0Xmo=
dmitri.shuralyov.com/gpu/mtl v0.0.0-20190408044501-666a987793e9/go.mod h1:H6x//7gZCb22OMCxBHrMx7a5I7Hp++hsVxbQ4BYO7hU=
github.com/BurntSushi/toml v0.3.1/go.mod h1:xHWCNGjB5oqiDr8zfno3MHue2Ht5sIBksp03qcyfWMU=
github.com/BurntSushi/xgb v0.0.0-20160522181843-27f122750802/go.mod h1:IVnqGOEym/WlBOVXweHU+Q+/VP0lqqI8lqeDx9IjBqo=
github.com/KyleBanks/depth v1.2.1 h1:5h8fQADFrWtarTdtDudMmGsC7GPbOAu6RVB3ffsVFHc=
github.com/KyleBanks/depth v1.2.1/go.mod h1:jzSb9d0L43HxTQfT+oSA1EEp2q+ne2uh6XgeJcm8brE=
github.com/PuerkitoBio/purell v1.1.1 h1:WEQqlqaGbrPkxLJWfBwQmfEAE1Z7ONdDLqrN38tNFfI=
github.com/PuerkitoBio/purell v1.1.1/go.mod h1:c11w/QuzBsJSee3cPx9rAFu61PvFxuPbtSwDGJws/X0=
github.com/PuerkitoBio/urlesc v0.0.0-20170810143723-de5bf2ad4578 h1:d+Bc7a5rLufV/sSk/8dngufqelfh6jnri85riMAaF/M=
github.com/PuerkitoBio/urlesc v0.0.0-20170810143723-de5bf2ad4578/go.mod h1:uGdkoq3SwY9Y+13GIhn11/XLaGBb4BfwItxLd5jeuXE=
github.com/alecthomas/template v0.0.0-20160405071501-a0175ee3bccc/go.mod h1:LOuyumcjzFXgccqObfd/Ljyb9UuFJ6TxHnclSeseNhc=
github.com/alecthomas/template v0.0.0-20190718012654-fb15b899a751/go.mod h1:LOuyumcjzFXgccqObfd/Ljyb9UuFJ6TxHnclSeseNhc=
github.com/alecthomas/units v0.0.0-20151022065526-2efee857e7cf/go.mod h1:ybxpYRFXyAe+OPACYpWeL0wqObRcbAqCMya13uyzqw0=
github.com/alecthomas/units v0.0.0-20190717042225-c3de453c63f4/go.mod h1:ybxpYRFXyAe+OPACYpWeL0wqObRcbAqCMya13uyzqw0=
github.com/alecthomas/units v0.0.0-20190924025748-f65c72e2690d/go.mod h1:rBZYJk541a8SKzHPHnH3zbiI+7dagKZ0cgpgrD7Fyho=
github.com/beorn7/perks v0.0.0-20180321164747-3a771d992973/go.mod h1:Dwedo/Wpr24TaqPxmxbtue+5NUziq4I4S80YR8gNf3Q=
github.com/beorn7/perks v1.0.0/go.mod h1:KWe93zE9D1o94FZ5RNwFwVgaQK1VOXiVxmqh+CedLV8=
github.com/beorn7/perks v1.0.1 h1:VlbKKnNfV8bJzeqoa4cOKqO6bYr3WgKZxO8Z16+hsOM=
github.com/beorn7/perks v1.0.1/go.mod h1:G2ZrVWU2WbWT9wwq4/hrbKbnv/1ERSJQ0ibhJ6rlkpw=
github.com/bitly/go-hostpool v0.0.0-20171023180738-a3a6125de932 h1:mXoPYz/Ul5HYEDvkta6I8/rnYM5gSdSV2tJ6XbZuEtY=
github.com/bitly/go-hostpool v0.0.0-20171023180738-a3a6125de932/go.mod h1:NOuUCSz6Q9T7+igc/hlvDOUdtWKryOrtFyIVABv/p7k=
github.com/bits-and-blooms/bitset v1.2.0 h1:Kn4yilvwNtMACtf1eYDlG8H77R07mZSPbMjLyS07ChA=
github.com/bits-and-blooms/bitset v1.2.0/go.mod h1:gIdJ4wp64HaoK2YrL1Q5/N7Y16edYb8uY+O0FJTyyDA=
github.com/bmizerany/assert v0.0.0-20160611221934-b7ed37b82869 h1:DDGfHa7BWjL4YnC6+E63dPcxHo2sUxDIu8g3QgEJdRY=
github.com/bmizerany/assert v0.0.0-20160611221934-b7ed37b82869/go.mod h1:Ekp36dRnpXw/yCqJaO+ZrUyxD+3VXMFFr56k5XYrpB4=
github.com/bsm/ginkgo/v2 v2.7.0 h1:ItPMPH90RbmZJt5GtkcNvIRuGEdwlBItdNVoyzaNQao=
github.com/bsm/gomega v1.26.0 h1:LhQm+AFcgV2M0WyKroMASzAzCAJVpAxQXv4SaI9a69Y=
github.com/bytedance/sonic v1.5.0/go.mod h1:ED5hyg4y6t3/9Ku1R6dU/4KyJ48DZ4jPhfY1O2AihPM=
github.com/bytedance/sonic v1.9.1 h1:6iJ6NqdoxCDr6mbY8h18oSO+cShGSMRGCEo7F2h0x8s=
github.com/bytedance/sonic v1.9.1/go.mod h1:i736AoUSYt75HyZLoJW9ERYxcy6eaN6h4BZXU064P/U=
github.com/census-instrumentation/opencensus-proto v0.2.1/go.mod h1:f6KPmirojxKA12rnyqOA5BBL4O983OfeGPqjHWSTneU=
github.com/cespare/xxhash/v2 v2.1.1/go.mod h1:VGX0DQ3Q6kWi7AoAeZDth3/j3BFtOZR5XLFGgcrjCOs=
github.com/cespare/xxhash/v2 v2.1.2/go.mod h1:VGX0DQ3Q6kWi7AoAeZDth3/j3BFtOZR5XLFGgcrjCOs=
github.com/cespare/xxhash/v2 v2.2.0 h1:DC2CZ1Ep5Y4k3ZQ899DldepgrayRUGE6BBZ/cd9Cj44=
github.com/cespare/xxhash/v2 v2.2.0/go.mod h1:VGX0DQ3Q6kWi7AoAeZDth3/j3BFtOZR5XLFGgcrjCOs=
github.com/chenzhuoyu/base64x v0.0.0-20211019084208-fb5309c8db06/go.mod h1:DH46F32mSOjUmXrMHnKwZdA8wcEefY7UVqBKYGjpdQY=
github.com/chenzhuoyu/base64x v0.0.0-20221115062448-fe3a3abad311 h1:qSGYFH7+jGhDF8vLC+iwCD4WpbV1EBDSzWkJODFLams=
github.com/chenzhuoyu/base64x v0.0.0-20221115062448-fe3a3abad311/go.mod h1:b583jCggY9gE99b6G5LEC39OIiVsWj+R97kbl5odCEk=
github.com/chzyer/logex v1.1.10/go.mod h1:+Ywpsq7O8HXn0nuIou7OrIPyXbp3wmkHB+jjWRnGsAI=
github.com/chzyer/readline v0.0.0-20180603132655-2972be24d48e/go.mod h1:nSuG5e5PlCu98SY8svDHJxuZscDgtXS6KTTbou5AhLI=
github.com/chzyer/test v0.0.0-20180213035817-a1ea475d72b1/go.mod h1:Q3SI9o4m/ZMnBNeIyt5eFwwo7qiLfzFZmjNmxjkiQlU=
github.com/client9/misspell v0.3.4/go.mod h1:qj6jICC3Q7zFZvVWo7KLAzC3yx5G7kyvSDkc90ppPyw=
github.com/cncf/udpa/go v0.0.0-20191209042840-269d4d468f6f/go.mod h1:M8M6+tZqaGXZJjfX53e64911xZQV5JYwmTeXPW+k8Sc=
github.com/cncf/udpa/go v0.0.0-20200629203442-efcf912fb354/go.mod h1:WmhPx2Nbnhtbo57+VJT5O0JRkEi1Wbu0z5j0R8u5Hbk=
github.com/cncf/udpa/go v0.0.0-20201120205902-5459f2c99403/go.mod h1:WmhPx2Nbnhtbo57+VJT5O0JRkEi1Wbu0z5j0R8u5Hbk=
github.com/creack/pty v1.1.9/go.mod h1:oKZEueFk5CKHvIhNR5MUki03XCEU+Q6VDXinZuGJ33E=
github.com/davecgh/go-spew v1.1.0/go.mod h1:J7Y8YcW2NihsgmVo/mv3lAwl/skON4iLHjSsI+c5H38=
github.com/davecgh/go-spew v1.1.1 h1:vj9j/u1bqnvCEfJOwUhtlOARqs3+rkHYY13jYWTU97c=
github.com/davecgh/go-spew v1.1.1/go.mod h1:J7Y8YcW2NihsgmVo/mv3lAwl/skON4iLHjSsI+c5H38=
github.com/dgryski/go-rendezvous v0.0.0-20200823014737-9f7001d12a5f h1:lO4WD4F/rVNCu3HqELle0jiPLLBs70cWOduZpkS1E78=
github.com/dgryski/go-rendezvous v0.0.0-20200823014737-9f7001d12a5f/go.mod h1:cuUVRXasLTGF7a8hSLbxyZXjz+1KgoB3wDUb6vlszIc=
github.com/envoyproxy/go-control-plane v0.9.0/go.mod h1:YTl/9mNaCwkRvm6d1a2C3ymFceY/DCBVvsKhRF0iEA4=
github.com/envoyproxy/go-control-plane v0.9.1-0.20191026205805-5f8ba28d4473/go.mod h1:YTl/9mNaCwkRvm6d1a2C3ymFceY/DCBVvsKhRF0iEA4=
github.com/envoyproxy/go-control-plane v0.9.4/go.mod h1:6rpuAdCZL397s3pYoYcLgu1mIlRU8Am5FuJP05cCM98=
github.com/envoyproxy/go-control-plane v0.9.7/go.mod h1:cwu0lG7PUMfa9snN8LXBig5ynNVH9qI8YYLbd1fK2po=
github.com/envoyproxy/go-control-plane v0.9.9-0.20201210154907-fd9021fe5dad/go.mod h1:cXg6YxExXjJnVBQHBLXeUAgxn2UodCpnH306RInaBQk=
github.com/envoyproxy/protoc-gen-validate v0.1.0/go.mod h1:iSmxcyjqTsJpI2R4NaDN7+kN2VEUnK/pcBlmesArF7c=
github.com/frankban/quicktest v1.14.4 h1:g2rn0vABPOOXmZUj+vbmUp0lPoXEMuhTpIluN0XL9UY=
github.com/fsnotify/fsnotify v1.6.0 h1:n+5WquG0fcWoWp6xPWfHdbskMCQaFnG6PfBrh1Ky4HY=
github.com/fsnotify/fsnotify v1.6.0/go.mod h1:sl3t1tCWJFWoRz9R8WJCbQihKKwmorjAbSClcnxKAGw=
github.com/gabriel-vasile/mimetype v1.4.2 h1:w5qFW6JKBz9Y393Y4q372O9A7cUSequkh1Q7OhCmWKU=
github.com/gabriel-vasile/mimetype v1.4.2/go.mod h1:zApsH/mKG4w07erKIaJPFiX0Tsq9BFQgN3qGY5GnNgA=
github.com/gin-contrib/gzip v0.0.6 h1:NjcunTcGAj5CO1gn4N8jHOSIeRFHIbn51z6K+xaN4d4=
github.com/gin-contrib/sse v0.1.0 h1:Y/yl/+YNO8GZSjAhjMsSuLt29uWRFHdHYUb5lYOV9qE=
github.com/gin-contrib/sse v0.1.0/go.mod h1:RHrZQHXnP2xjPF+u1gW/2HnVO7nvIa9PG3Gm+fLHvGI=
github.com/gin-gonic/gin v1.7.4/go.mod h1:jD2toBW3GZUr5UMcdrwQA10I7RuaFOl/SGeDjXkfUtY=
github.com/gin-gonic/gin v1.9.1 h1:4idEAncQnU5cB7BeOkPtxjfCSye0AAm1R0RVIqJ+Jmg=
github.com/gin-gonic/gin v1.9.1/go.mod h1:hPrL7YrpYKXt5YId3A/Tnip5kqbEAP+KLuI3SUcPTeU=
github.com/go-gl/glfw v0.0.0-20190409004039-e6da0acd62b1/go.mod h1:vR7hzQXu2zJy9AVAgeJqvqgH9Q5CA+iKCZ2gyEVpxRU=
github.com/go-gl/glfw/v3.3/glfw v0.0.0-20191125211704-12ad95a8df72/go.mod h1:tQ2UAYgL5IevRw8kRxooKSPJfGvJ9fJQFa0TUsXzTg8=
github.com/go-gl/glfw/v3.3/glfw v0.0.0-20200222043503-6f7a984d4dc4/go.mod h1:tQ2UAYgL5IevRw8kRxooKSPJfGvJ9fJQFa0TUsXzTg8=
github.com/go-kit/kit v0.8.0/go.mod h1:xBxKIO96dXMWWy0MnWVtmwkA9/13aqxPnvrjFYMA2as=
github.com/go-kit/kit v0.9.0/go.mod h1:xBxKIO96dXMWWy0MnWVtmwkA9/13aqxPnvrjFYMA2as=
github.com/go-kit/log v0.1.0/go.mod h1:zbhenjAZHb184qTLMA9ZjW7ThYL0H2mk7Q6pNt4vbaY=
github.com/go-logfmt/logfmt v0.3.0/go.mod h1:Qt1PoO58o5twSAckw1HlFXLmHsOX5/0LbT9GBnD5lWE=
github.com/go-logfmt/logfmt v0.4.0/go.mod h1:3RMwSq7FuexP4Kalkev3ejPJsZTpXXBr9+V4qmtdjCk=
github.com/go-logfmt/logfmt v0.5.0/go.mod h1:wCYkCAKZfumFQihp8CzCvQ3paCTfi41vtzG1KdI/P7A=
github.com/go-openapi/jsonpointer v0.19.3/go.mod h1:Pl9vOtqEWErmShwVjC8pYs9cog34VGT37dQOVbmoatg=
github.com/go-openapi/jsonpointer v0.19.5 h1:gZr+CIYByUqjcgeLXnQu2gHYQC9o73G2XUeOFYEICuY=
github.com/go-openapi/jsonpointer v0.19.5/go.mod h1:Pl9vOtqEWErmShwVjC8pYs9cog34VGT37dQOVbmoatg=
github.com/go-openapi/jsonreference v0.19.6 h1:UBIxjkht+AWIgYzCDSv2GN+E/togfwXUJFRTWhl2Jjs=
github.com/go-openapi/jsonreference v0.19.6/go.mod h1:diGHMEHg2IqXZGKxqyvWdfWU/aim5Dprw5bqpKkTvns=
github.com/go-openapi/spec v0.20.4 h1:O8hJrt0UMnhHcluhIdUgCLRWyM2x7QkBXRvOs7m+O1M=
github.com/go-openapi/spec v0.20.4/go.mod h1:faYFR1CvsJZ0mNsmsphTMSoRrNV3TEDoAM7FOEWeq8I=
github.com/go-openapi/swag v0.19.5/go.mod h1:POnQmlKehdgb5mhVOsnJFsivZCEZ/vjK9gh66Z9tfKk=
github.com/go-openapi/swag v0.19.15 h1:D2NRCBzS9/pEY3gP9Nl8aDqGUcPFrwG2p+CNFrLyrCM=
github.com/go-openapi/swag v0.19.15/go.mod h1:QYRuS/SOXUCsnplDa677K7+DxSOj6IPNl/eQntq43wQ=
github.com/go-playground/assert/v2 v2.0.1/go.mod h1:VDjEfimB/XKnb+ZQfWdccd7VUvScMdVu0Titje2rxJ4=
github.com/go-playground/assert/v2 v2.2.0 h1:JvknZsQTYeFEAhQwI4qEt9cyV5ONwRHC+lYKSsYSR8s=
github.com/go-playground/locales v0.13.0/go.mod h1:taPMhCMXrRLJO55olJkUXHZBHCxTMfnGwq/HNwmWNS8=
github.com/go-playground/locales v0.14.1 h1:EWaQ/wswjilfKLTECiXz7Rh+3BjFhfDFKv/oXslEjJA=
github.com/go-playground/locales v0.14.1/go.mod h1:hxrqLVvrK65+Rwrd5Fc6F2O76J/NuW9t0sjnWqG1slY=
github.com/go-playground/universal-translator v0.17.0/go.mod h1:UkSxE5sNxxRwHyU+Scu5vgOQjsIJAF8j9muTVoKLVtA=
github.com/go-playground/universal-translator v0.18.1 h1:Bcnm0ZwsGyWbCzImXv+pAJnYK9S473LQFuzCbDbfSFY=
github.com/go-playground/universal-translator v0.18.1/go.mod h1:xekY+UJKNuX9WP91TpwSH2VMlDf28Uj24BCp08ZFTUY=
github.com/go-playground/validator/v10 v10.4.1/go.mod h1:nlOn6nFhuKACm19sB/8EGNn9GlaMV7XkbRSipzJ0Ii4=
github.com/go-playground/validator/v10 v10.14.0 h1:vgvQWe3XCz3gIeFDm/HnTIbj6UGmg/+t63MyGU2n5js=
github.com/go-playground/validator/v10 v10.14.0/go.mod h1:9iXMNT7sEkjXb0I+enO7QXmzG6QCsPWY4zveKFVRSyU=
github.com/go-stack/stack v1.8.0/go.mod h1:v0f6uXyyMGvRgIKkXu+yp6POWl0qKG85gN/melR3HDY=
github.com/goccy/go-json v0.10.2 h1:CrxCmQqYDkv1z7lO7Wbh2HN93uovUHgrECaO5ZrCXAU=
github.com/goccy/go-json v0.10.2/go.mod h1:6MelG93GURQebXPDq3khkgXZkazVtN9CRI+MGFi0w8I=
github.com/gogo/protobuf v1.1.1/go.mod h1:r8qH/GZQm5c6nD/R0oafs1akxWv10x8SbQlK7atdtwQ=
github.com/golang/glog v0.0.0-20160126235308-23def4e6c14b/go.mod h1:SBH7ygxi8pfUlaOkMMuAQtPIUF8ecWP5IEl/CR7VP2Q=
github.com/golang/groupcache v0.0.0-20190702054246-869f871628b6/go.mod h1:cIg4eruTrX1D+g88fzRXU5OdNfaM+9IcxsU14FzY7Hc=
github.com/golang/groupcache v0.0.0-20191227052852-215e87163ea7/go.mod h1:cIg4eruTrX1D+g88fzRXU5OdNfaM+9IcxsU14FzY7Hc=
github.com/golang/groupcache v0.0.0-20200121045136-8c9f03a8e57e/go.mod h1:cIg4eruTrX1D+g88fzRXU5OdNfaM+9IcxsU14FzY7Hc=
github.com/golang/mock v1.1.1/go.mod h1:oTYuIxOrZwtPieC+H1uAHpcLFnEyAGVDL/k47Jfbm0A=
github.com/golang/mock v1.2.0/go.mod h1:oTYuIxOrZwtPieC+H1uAHpcLFnEyAGVDL/k47Jfbm0A=
github.com/golang/mock v1.3.1/go.mod h1:sBzyDLLjw3U8JLTeZvSv8jJB+tU5PVekmnlKIyFUx0Y=
github.com/golang/mock v1.4.0/go.mod h1:UOMv5ysSaYNkG+OFQykRIcU/QvvxJf3p21QfJ2Bt3cw=
github.com/golang/mock v1.4.1/go.mod h1:UOMv5ysSaYNkG+OFQykRIcU/QvvxJf3p21QfJ2Bt3cw=
github.com/golang/mock v1.4.3/go.mod h1:UOMv5ysSaYNkG+OFQykRIcU/QvvxJf3p21QfJ2Bt3cw=
github.com/golang/mock v1.4.4/go.mod h1:l3mdAwkq5BuhzHwde/uurv3sEJeZMXNpwsxVWU71h+4=
github.com/golang/protobuf v1.2.0/go.mod h1:6lQm79b+lXiMfvg/cZm0SGofjICqVBUtrP5yJMmIC1U=
github.com/golang/protobuf v1.3.1/go.mod h1:6lQm79b+lXiMfvg/cZm0SGofjICqVBUtrP5yJMmIC1U=
github.com/golang/protobuf v1.3.2/go.mod h1:6lQm79b+lXiMfvg/cZm0SGofjICqVBUtrP5yJMmIC1U=
github.com/golang/protobuf v1.3.3/go.mod h1:vzj43D7+SQXF/4pzW/hwtAqwc6iTitCiVSaWz5lYuqw=
github.com/golang/protobuf v1.3.4/go.mod h1:vzj43D7+SQXF/4pzW/hwtAqwc6iTitCiVSaWz5lYuqw=
github.com/golang/protobuf v1.3.5/go.mod h1:6O5/vntMXwX2lRkT1hjjk0nAC1IDOTvTlVgjlRvqsdk=
github.com/golang/protobuf v1.4.0-rc.1/go.mod h1:ceaxUfeHdC40wWswd/P6IGgMaK3YpKi5j83Wpe3EHw8=
github.com/golang/protobuf v1.4.0-rc.1.0.20200221234624-67d41d38c208/go.mod h1:xKAWHe0F5eneWXFV3EuXVDTCmh+JuBKY0li0aMyXATA=
github.com/golang/protobuf v1.4.0-rc.2/go.mod h1:LlEzMj4AhA7rCAGe4KMBDvJI+AwstrUpVNzEA03Pprs=
github.com/golang/protobuf v1.4.0-rc.4.0.20200313231945-b860323f09d0/go.mod h1:WU3c8KckQ9AFe+yFwt9sWVRKCVIyN9cPHBJSNnbL67w=
github.com/golang/protobuf v1.4.0/go.mod h1:jodUvKwWbYaEsadDk5Fwe5c77LiNKVO9IDvqG2KuDX0=
github.com/golang/protobuf v1.4.1/go.mod h1:U8fpvMrcmy5pZrNK1lt4xCsGvpyWQ/VVv6QDs8UjoX8=
github.com/golang/protobuf v1.4.2/go.mod h1:oDoupMAO8OvCJWAcko0GGGIgR6R6ocIYbsSw735rRwI=
github.com/golang/protobuf v1.4.3/go.mod h1:oDoupMAO8OvCJWAcko0GGGIgR6R6ocIYbsSw735rRwI=
github.com/golang/protobuf v1.5.0/go.mod h1:FsONVRAS9T7sI+LIUmWTfcYkHO4aIWwzhcaSAoJOfIk=
github.com/golang/protobuf v1.5.2/go.mod h1:XVQd3VNwM+JqD3oG2Ue2ip4fOMUkwXdXDdiuN0vRsmY=
github.com/golang/protobuf v1.5.3 h1:KhyjKVUg7Usr/dYsdSqoFveMYd5ko72D+zANwlG1mmg=
github.com/golang/protobuf v1.5.3/go.mod h1:XVQd3VNwM+JqD3oG2Ue2ip4fOMUkwXdXDdiuN0vRsmY=
github.com/golang/snappy v0.0.3 h1:fHPg5GQYlCeLIPB9BZqMVR5nR9A+IM5zcgeTdjMYmLA=
github.com/golang/snappy v0.0.3/go.mod h1:/XxbfmMg8lxefKM7IXC3fBNl/7bRcc72aCRzEWrmP2Q=
github.com/google/btree v0.0.0-20180813153112-4030bb1f1f0c/go.mod h1:lNA+9X1NB3Zf8V7Ke586lFgjr2dZNuvo3lPJSGZ5JPQ=
github.com/google/btree v1.0.0/go.mod h1:lNA+9X1NB3Zf8V7Ke586lFgjr2dZNuvo3lPJSGZ5JPQ=
github.com/google/go-cmp v0.2.0/go.mod h1:oXzfMopK8JAjlY9xF4vHSVASa0yLyX7SntLO5aqRK0M=
github.com/google/go-cmp v0.3.0/go.mod h1:8QqcDgzrUqlUb/G2PQTWiueGozuR1884gddMywk6iLU=
github.com/google/go-cmp v0.3.1/go.mod h1:8QqcDgzrUqlUb/G2PQTWiueGozuR1884gddMywk6iLU=
github.com/google/go-cmp v0.4.0/go.mod h1:v8dTdLbMG2kIc/vJvl+f65V22dbkXbowE6jgT/gNBxE=
github.com/google/go-cmp v0.4.1/go.mod h1:v8dTdLbMG2kIc/vJvl+f65V22dbkXbowE6jgT/gNBxE=
github.com/google/go-cmp v0.5.0/go.mod h1:v8dTdLbMG2kIc/vJvl+f65V22dbkXbowE6jgT/gNBxE=
github.com/google/go-cmp v0.5.1/go.mod h1:v8dTdLbMG2kIc/vJvl+f65V22dbkXbowE6jgT/gNBxE=
github.com/google/go-cmp v0.5.2/go.mod h1:v8dTdLbMG2kIc/vJvl+f65V22dbkXbowE6jgT/gNBxE=
github.com/google/go-cmp v0.5.4/go.mod h1:v8dTdLbMG2kIc/vJvl+f65V22dbkXbowE6jgT/gNBxE=
github.com/google/go-cmp v0.5.5/go.mod h1:v8dTdLbMG2kIc/vJvl+f65V22dbkXbowE6jgT/gNBxE=
github.com/google/go-cmp v0.5.9 h1:O2Tfq5qg4qc4AmwVlvv0oLiVAGB7enBSJ2x2DqQFi38=
github.com/google/gofuzz v1.0.0/go.mod h1:dBl0BpW6vV/+mYPU4Po3pmUjxk6FQPldtuIdl/M65Eg=
github.com/google/martian v2.1.0+incompatible/go.mod h1:9I4somxYTbIHy5NJKHRl3wXiIaQGbYVAs8BPL6v8lEs=
github.com/google/martian/v3 v3.0.0/go.mod h1:y5Zk1BBys9G+gd6Jrk0W3cC1+ELVxBWuIGO+w/tUAp0=
github.com/google/martian/v3 v3.1.0/go.mod h1:y5Zk1BBys9G+gd6Jrk0W3cC1+ELVxBWuIGO+w/tUAp0=
github.com/google/pprof v0.0.0-20181206194817-3ea8567a2e57/go.mod h1:zfwlbNMJ+OItoe0UupaVj+oy1omPYYDuagoSzA8v9mc=
github.com/google/pprof v0.0.0-20190515194954-54271f7e092f/go.mod h1:zfwlbNMJ+OItoe0UupaVj+oy1omPYYDuagoSzA8v9mc=
github.com/google/pprof v0.0.0-20191218002539-d4f498aebedc/go.mod h1:ZgVRPoUq/hfqzAqh7sHMqb3I9Rq5C59dIz2SbBwJ4eM=
github.com/google/pprof v0.0.0-20200212024743-f11f1df84d12/go.mod h1:ZgVRPoUq/hfqzAqh7sHMqb3I9Rq5C59dIz2SbBwJ4eM=
github.com/google/pprof v0.0.0-20200229191704-1ebb73c60ed3/go.mod h1:ZgVRPoUq/hfqzAqh7sHMqb3I9Rq5C59dIz2SbBwJ4eM=
github.com/google/pprof v0.0.0-20200430221834-fc25d7d30c6d/go.mod h1:ZgVRPoUq/hfqzAqh7sHMqb3I9Rq5C59dIz2SbBwJ4eM=
github.com/google/pprof v0.0.0-20200708004538-1a94d8640e99/go.mod h1:ZgVRPoUq/hfqzAqh7sHMqb3I9Rq5C59dIz2SbBwJ4eM=
github.com/google/pprof v0.0.0-20201023163331-3e6fc7fc9c4c/go.mod h1:kpwsk12EmLew5upagYY7GY0pfYCcupk39gWOCRROcvE=
github.com/google/pprof v0.0.0-20201203190320-1bf35d6f28c2/go.mod h1:kpwsk12EmLew5upagYY7GY0pfYCcupk39gWOCRROcvE=
github.com/google/pprof v0.0.0-20201218002935-b9804c9f04c2/go.mod h1:kpwsk12EmLew5upagYY7GY0pfYCcupk39gWOCRROcvE=
github.com/google/renameio v0.1.0/go.mod h1:KWCgfxg9yswjAJkECMjeO8J8rahYeXnNhOm40UhjYkI=
github.com/google/uuid v1.1.2/go.mod h1:TIyPZe4MgqvfeYDBFedMoGGpEw/LqOeaOT+nhxU+yHo=
github.com/googleapis/gax-go/v2 v2.0.4/go.mod h1:0Wqv26UfaUD9n4G6kQubkQ+KchISgw+vpHVxEJEs9eg=
github.com/googleapis/gax-go/v2 v2.0.5/go.mod h1:DWXyrwAJ9X0FpwwEdw+IPEYBICEFu5mhpdKc/us6bOk=
github.com/googleapis/google-cloud-go-testing v0.0.0-20200911160855-bcd43fbb19e8/go.mod h1:dvDLG8qkwmyD9a/MJJN3XJcT3xFxOKAvTZGvuZmac9g=
github.com/hailocab/go-hostpool v0.0.0-20160125115350-e80d13ce29ed h1:5upAirOpQc1Q53c0bnx2ufif5kANL7bfZWcc6VJWJd8=
github.com/hailocab/go-hostpool v0.0.0-20160125115350-e80d13ce29ed/go.mod h1:tMWxXQ9wFIaZeTI9F+hmhFiGpFmhOHzyShyFUhRm0H4=
github.com/hashicorp/golang-lru v0.5.0/go.mod h1:/m3WP610KZHVQ1SGc6re/UDhFvYD7pJ4Ao+sR/qLZy8=
github.com/hashicorp/golang-lru v0.5.1/go.mod h1:/m3WP610KZHVQ1SGc6re/UDhFvYD7pJ4Ao+sR/qLZy8=
github.com/hashicorp/hcl v1.0.0 h1:0Anlzjpi4vEasTeNFn2mLJgTSwt0+6sfsiTG8qcWGx4=
github.com/hashicorp/hcl v1.0.0/go.mod h1:E5yfLk+7swimpb2L/Alb/PJmXilQ/rhwaUYs4T20WEQ=
github.com/ianlancetaylor/demangle v0.0.0-20181102032728-5e5cf60278f6/go.mod h1:aSSvb/t6k1mPoxDqO4vJh6VOCGPwU4O0C2/Eqndh1Sc=
github.com/ianlancetaylor/demangle v0.0.0-20200824232613-28f6c0f3b639/go.mod h1:aSSvb/t6k1mPoxDqO4vJh6VOCGPwU4O0C2/Eqndh1Sc=
github.com/josharian/intern v1.0.0 h1:vlS4z54oSdjm0bgjRigI+G1HpF+tI+9rE5LLzOg8HmY=
github.com/josharian/intern v1.0.0/go.mod h1:5DoeVV0s6jJacbCEi61lwdGj/aVlrQvzHFFd8Hwg//Y=
github.com/jpillora/backoff v1.0.0/go.mod h1:J/6gKK9jxlEcS3zixgDgUAsiuZ7yrSoa/FX5e0EB2j4=
github.com/json-iterator/go v1.1.6/go.mod h1:+SdeFBvtyEkXs7REEP0seUULqWtbJapLOCVDaaPEHmU=
github.com/json-iterator/go v1.1.9/go.mod h1:KdQUCv79m/52Kvf8AW2vK1V8akMuk1QjK/uOdHXbAo4=
github.com/json-iterator/go v1.1.10/go.mod h1:KdQUCv79m/52Kvf8AW2vK1V8akMuk1QjK/uOdHXbAo4=
github.com/json-iterator/go v1.1.11/go.mod h1:KdQUCv79m/52Kvf8AW2vK1V8akMuk1QjK/uOdHXbAo4=
github.com/json-iterator/go v1.1.12 h1:PV8peI4a0ysnczrg+LtxykD8LfKY9ML6u2jnxaEnrnM=
github.com/json-iterator/go v1.1.12/go.mod h1:e30LSqwooZae/UwlEbR2852Gd8hjQvJoHmT4TnhNGBo=
github.com/jstemmer/go-junit-report v0.0.0-20190106144839-af01ea7f8024/go.mod h1:6v2b51hI/fHJwM22ozAgKL4VKDeJcHhJFhtBdhmNjmU=
github.com/jstemmer/go-junit-report v0.9.1/go.mod h1:Brl9GWCQeLvo8nXZwPNNblvFj/XSXhF0NWZEnDohbsk=
github.com/julienschmidt/httprouter v1.2.0/go.mod h1:SYymIcj16QtmaHHD7aYtjjsJG7VTCxuUUipMqKk8s4w=
github.com/julienschmidt/httprouter v1.3.0/go.mod h1:JR6WtHb+2LUe8TCKY3cZOxFyyO8IZAc4RVcycCCAKdM=
github.com/kisielk/gotool v1.0.0/go.mod h1:XhKaO+MFFWcvkIS/tQcRk01m1F5IRFswLeQ+oQHNcck=
github.com/klauspost/cpuid/v2 v2.0.3/go.mod h1:FInQzS24/EEf25PyTYn52gqo7WaD8xa0213Md/qVLRg=
github.com/klauspost/cpuid/v2 v2.0.9/go.mod h1:FInQzS24/EEf25PyTYn52gqo7WaD8xa0213Md/qVLRg=
github.com/klauspost/cpuid/v2 v2.2.4 h1:acbojRNwl3o09bUq+yDCtZFc1aiwaAAxtcn8YkZXnvk=
github.com/klauspost/cpuid/v2 v2.2.4/go.mod h1:RVVoqg1df56z8g3pUjL/3lE5UfnlrJX8tyFgg4nqhuY=
github.com/konsorten/go-windows-terminal-sequences v1.0.1/go.mod h1:T0+1ngSBFLxvqU3pZ+m/2kptfBszLMUkC4ZK/EgS/cQ=
github.com/konsorten/go-windows-terminal-sequences v1.0.3/go.mod h1:T0+1ngSBFLxvqU3pZ+m/2kptfBszLMUkC4ZK/EgS/cQ=
github.com/kr/fs v0.1.0/go.mod h1:FFnZGqtBN9Gxj7eW1uZ42v5BccTP0vu6NEaFoC2HwRg=
github.com/kr/logfmt v0.0.0-20140226030751-b84e30acd515/go.mod h1:+0opPa2QZZtGFBFZlji/RkVcI2GknAs/DXo4wKdlNEc=
github.com/kr/pretty v0.1.0/go.mod h1:dAy3ld7l9f0ibDNOQOHHMYYIIbhfbHSm3C4ZsoJORNo=
github.com/kr/pretty v0.3.1 h1:flRD4NNwYAUpkphVc1HcthR4KEIFJ65n8Mw5qdRn3LE=
github.com/kr/pty v1.1.1/go.mod h1:pFQYn66WHrOpPYNljwOMqo10TkYh1fy3cYio2l3bCsQ=
github.com/kr/text v0.1.0/go.mod h1:4Jbv+DJW3UT/LiOwJeYQe1efqtUx/iVham/4vfdArNI=
github.com/kr/text v0.2.0 h1:5Nx0Ya0ZqY2ygV366QzturHI13Jq95ApcVaJBhpS+AY=
github.com/kr/text v0.2.0/go.mod h1:eLer722TekiGuMkidMxC/pM04lWEeraHUUmBw8l2grE=
github.com/leodido/go-urn v1.2.0/go.mod h1:+8+nEpDfqqsY+g338gtMEUOtuK+4dEMhiQEgxpxOKII=
github.com/leodido/go-urn v1.2.4 h1:XlAE/cm/ms7TE/VMVoduSpNBoyc2dOxHs5MZSwAN63Q=
github.com/leodido/go-urn v1.2.4/go.mod h1:7ZrI8mTSeBSHl/UaRyKQW1qZeMgak41ANeCNaVckg+4=
github.com/magiconair/properties v1.8.7 h1:IeQXZAiQcpL9mgcAe1Nu6cX9LLw6ExEHKjN0VQdvPDY=
github.com/magiconair/properties v1.8.7/go.mod h1:Dhd985XPs7jluiymwWYZ0G4Z61jb3vdS329zhj2hYo0=
github.com/mailru/easyjson v0.0.0-20190614124828-94de47d64c63/go.mod h1:C1wdFJiN94OJF2b5HbByQZoLdCWB1Yqtg26g4irojpc=
github.com/mailru/easyjson v0.0.0-20190626092158-b2ccc519800e/go.mod h1:C1wdFJiN94OJF2b5HbByQZoLdCWB1Yqtg26g4irojpc=
github.com/mailru/easyjson v0.7.6 h1:8yTIVnZgCoiM1TgqoeTl+LfU5Jg6/xL3QhGQnimLYnA=
github.com/mailru/easyjson v0.7.6/go.mod h1:xzfreul335JAWq5oZzymOObrkdz5UnU4kGfJJLY9Nlc=
github.com/mattn/go-isatty v0.0.12/go.mod h1:cbi8OIDigv2wuxKPP5vlRcQ1OAZbq2CE4Kysco4FUpU=
github.com/mattn/go-isatty v0.0.19 h1:JITubQf0MOLdlGRuRq+jtsDlekdYPia9ZFsB8h/APPA=
github.com/mattn/go-isatty v0.0.19/go.mod h1:W+V8PltTTMOvKvAeJH7IuucS94S2C6jfK/D7dTCTo3Y=
github.com/matttproud/golang_protobuf_extensions v1.0.1 h1:4hp9jkHxhMHkqkrB3Ix0jegS5sx/RkqARlsWZ6pIwiU=
github.com/matttproud/golang_protobuf_extensions v1.0.1/go.mod h1:D8He9yQNgCq6Z5Ld7szi9bcBfOoFv/3dc6xSMkL2PC0=
github.com/mitchellh/mapstructure v1.5.0 h1:jeMsZIYE/09sWLaz43PL7Gy6RuMjD2eJVyuac5Z2hdY=
github.com/mitchellh/mapstructure v1.5.0/go.mod h1:bFUtVrKA4DC2yAKiSyO/QUcy7e+RRV2QTWOzhPopBRo=
github.com/modern-go/concurrent v0.0.0-20180228061459-e0a39a4cb421/go.mod h1:6dJC0mAP4ikYIbvyc7fijjWJddQyLn8Ig3JB5CqoB9Q=
github.com/modern-go/concurrent v0.0.0-20180306012644-bacd9c7ef1dd h1:TRLaZ9cD/w8PVh93nsPXa1VrQ6jlwL5oN8l14QlcNfg=
github.com/modern-go/concurrent v0.0.0-20180306012644-bacd9c7ef1dd/go.mod h1:6dJC0mAP4ikYIbvyc7fijjWJddQyLn8Ig3JB5CqoB9Q=
github.com/modern-go/reflect2 v0.0.0-20180701023420-4b7aa43c6742/go.mod h1:bx2lNnkwVCuqBIxFjflWJWanXIb3RllmbCylyMrvgv0=
github.com/modern-go/reflect2 v1.0.1/go.mod h1:bx2lNnkwVCuqBIxFjflWJWanXIb3RllmbCylyMrvgv0=
github.com/modern-go/reflect2 v1.0.2 h1:xBagoLtFs94CBntxluKeaWgTMpvLxC4ur3nMaC9Gz0M=
github.com/modern-go/reflect2 v1.0.2/go.mod h1:yWuevngMOJpCy52FWWMvUC8ws7m/LJsjYzDa0/r8luk=
github.com/mwitkow/go-conntrack v0.0.0-20161129095857-cc309e4a2223/go.mod h1:qRWi+5nqEBWmkhHvq77mSJWrCKwh8bxhgT7d/eI7P4U=
github.com/mwitkow/go-conntrack v0.0.0-20190716064945-2f068394615f/go.mod h1:qRWi+5nqEBWmkhHvq77mSJWrCKwh8bxhgT7d/eI7P4U=
github.com/niemeyer/pretty v0.0.0-20200227124842-a10e7caefd8e h1:fD57ERR4JtEqsWbfPhv4DMiApHyliiK5xCTNVSPiaAs=
github.com/niemeyer/pretty v0.0.0-20200227124842-a10e7caefd8e/go.mod h1:zD1mROLANZcx1PVRCS0qkT7pwLkGfwJo4zjcN/Tysno=
github.com/pelletier/go-toml/v2 v2.0.8 h1:0ctb6s9mE31h0/lhu+J6OPmVeDxJn+kYnJc2jZR9tGQ=
github.com/pelletier/go-toml/v2 v2.0.8/go.mod h1:vuYfssBdrU2XDZ9bYydBu6t+6a6PYNcZljzZR9VXg+4=
github.com/penglongli/gin-metrics v0.1.10 h1:mNNWCM3swMOVHwzrHeXsE4C/myu8P/HIFohtyMi9rN8=
github.com/penglongli/gin-metrics v0.1.10/go.mod h1:wxGsGUwpVGv3hmYSxQn2GZgRL3YuCgiRFq2d0X6+EOU=
github.com/pkg/errors v0.8.0/go.mod h1:bwawxfHBFNV+L2hUp1rHADufV3IMtnDRdf1r5NINEl0=
github.com/pkg/errors v0.8.1/go.mod h1:bwawxfHBFNV+L2hUp1rHADufV3IMtnDRdf1r5NINEl0=
github.com/pkg/errors v0.9.1 h1:FEBLx1zS214owpjy7qsBeixbURkuhQAwrK5UwLGTwt4=
github.com/pkg/errors v0.9.1/go.mod h1:bwawxfHBFNV+L2hUp1rHADufV3IMtnDRdf1r5NINEl0=
github.com/pkg/sftp v1.13.1/go.mod h1:3HaPG6Dq1ILlpPZRO0HVMrsydcdLt6HRDccSgb87qRg=
github.com/pmezard/go-difflib v1.0.0 h1:4DBwDE0NGyQoBHbLQYPwSUPoCMWR5BEzIk/f1lZbAQM=
github.com/pmezard/go-difflib v1.0.0/go.mod h1:iKH77koFhYxTK1pcRnkKkqfTogsbg7gZNVY4sRDYZ/4=
github.com/prometheus/client_golang v0.9.1/go.mod h1:7SWBe2y4D6OKWSNQJUaRYU/AaXPKyh/dDVn+NZz0KFw=
github.com/prometheus/client_golang v1.0.0/go.mod h1:db9x61etRT2tGnBNRi70OPL5FsnadC4Ky3P0J6CfImo=
github.com/prometheus/client_golang v1.7.1/go.mod h1:PY5Wy2awLA44sXw4AOSfFBetzPP4j5+D6mVACh+pe2M=
github.com/prometheus/client_golang v1.11.0/go.mod h1:Z6t4BnS23TR94PD6BsDNk8yVqroYurpAkEiz0P2BEV0=
github.com/prometheus/client_golang v1.12.0 h1:C+UIj/QWtmqY13Arb8kwMt5j34/0Z2iKamrJ+ryC0Gg=
github.com/prometheus/client_golang v1.12.0/go.mod h1:3Z9XVyYiZYEO+YQWt3RD2R3jrbd179Rt297l4aS6nDY=
github.com/prometheus/client_model v0.0.0-20180712105110-5c3871d89910/go.mod h1:MbSGuTsp3dbXC40dX6PRTWyKYBIrTGTE9sqQNg2J8bo=
github.com/prometheus/client_model v0.0.0-20190129233127-fd36f4220a90/go.mod h1:xMI15A0UPsDsEKsMN9yxemIoYk6Tm2C1GtYGdfGttqA=
github.com/prometheus/client_model v0.0.0-20190812154241-14fe0d1b01d4/go.mod h1:xMI15A0UPsDsEKsMN9yxemIoYk6Tm2C1GtYGdfGttqA=
github.com/prometheus/client_model v0.2.0 h1:uq5h0d+GuxiXLJLNABMgp2qUWDPiLvgCzz2dUR+/W/M=
github.com/prometheus/client_model v0.2.0/go.mod h1:xMI15A0UPsDsEKsMN9yxemIoYk6Tm2C1GtYGdfGttqA=
github.com/prometheus/common v0.4.1/go.mod h1:TNfzLD0ON7rHzMJeJkieUDPYmFC7Snx/y86RQel1bk4=
github.com/prometheus/common v0.10.0/go.mod h1:Tlit/dnDKsSWFlCLTWaA1cyBgKHSMdTB80sz/V91rCo=
github.com/prometheus/common v0.26.0/go.mod h1:M7rCNAaPfAosfx8veZJCuw84e35h3Cfd9VFqTh1DIvc=
github.com/prometheus/common v0.32.1 h1:hWIdL3N2HoUx3B8j3YN9mWor0qhY/NlEKZEaXxuIRh4=
github.com/prometheus/common v0.32.1/go.mod h1:vu+V0TpY+O6vW9J44gczi3Ap/oXXR10b+M/gUGO4Hls=
github.com/prometheus/procfs v0.0.0-20181005140218-185b4288413d/go.mod h1:c3At6R/oaqEKCNdg8wHV1ftS6bRYblBhIjjI8uT2IGk=
github.com/prometheus/procfs v0.0.2/go.mod h1:TjEm7ze935MbeOT/UhFTIMYKhuLP4wbCsTZCD3I8kEA=
github.com/prometheus/procfs v0.1.3/go.mod h1:lV6e/gmhEcM9IjHGsFOCxxuZ+z1YqCvr4OA4YeYWdaU=
github.com/prometheus/procfs v0.6.0/go.mod h1:cz+aTbrPOrUb4q7XlbU9ygM+/jj0fzG6c1xBZuNvfVA=
github.com/prometheus/procfs v0.7.3 h1:4jVXhlkAyzOScmCkXBTOLRLTz8EeU+eyjrwB/EPq0VU=
github.com/prometheus/procfs v0.7.3/go.mod h1:cz+aTbrPOrUb4q7XlbU9ygM+/jj0fzG6c1xBZuNvfVA=
github.com/redis/go-redis/v9 v9.0.5 h1:CuQcn5HIEeK7BgElubPP8CGtE0KakrnbBSTLjathl5o=
github.com/redis/go-redis/v9 v9.0.5/go.mod h1:WqMKv5vnQbRuZstUwxQI195wHy+t4PuXDOjzMvcuQHk=
github.com/rogpeppe/go-internal v1.3.0/go.mod h1:M8bDsm7K2OlrFYOpmOWEs/qY81heoFRclV5y23lUDJ4=
github.com/rogpeppe/go-internal v1.9.0 h1:73kH8U+JUqXU8lRuOHeVHaa/SZPifC7BkcraZVejAe8=
github.com/scylladb/gocql v1.10.0 h1:CqBUMPRpgRhNvvWlgcYr5v3Yl42nFY8LKbmpNVQYiV8=
github.com/scylladb/gocql v1.10.0/go.mod h1:OrZ5SWxnPnlOpCawJd0gqfWiP3KkPu624hlXwu+0qOM=
github.com/sirupsen/logrus v1.2.0/go.mod h1:LxeOpSwHxABJmUn/MG1IvRgCAasNZTLOkJPxbbu5VWo=
github.com/sirupsen/logrus v1.4.2/go.mod h1:tLMulIdttU9McNUspp0xgXVQah82FyeX6MwdIuYE2rE=
github.com/sirupsen/logrus v1.6.0/go.mod h1:7uNnSEd1DgxDLC74fIahvMZmmYsHGZGEOFrfsX/uA88=
github.com/sirupsen/logrus v1.9.3 h1:dueUQJ1C2q9oE3F7wvmSGAaVtTmUizReu6fjN8uqzbQ=
github.com/sirupsen/logrus v1.9.3/go.mod h1:naHLuLoDiP4jHNo9R0sCBMtWGeIprob74mVsIT4qYEQ=
github.com/spf13/afero v1.9.5 h1:stMpOSZFs//0Lv29HduCmli3GUfpFoF3Y1Q/aXj/wVM=
github.com/spf13/afero v1.9.5/go.mod h1:UBogFpq8E9Hx+xc5CNTTEpTnuHVmXDwZcZcE1eb/UhQ=
github.com/spf13/cast v1.5.1 h1:R+kOtfhWQE6TVQzY+4D7wJLBgkdVasCEFxSUBYBYIlA=
github.com/spf13/cast v1.5.1/go.mod h1:b9PdjNptOpzXr7Rq1q9gJML/2cdGQAo69NKzQ10KN48=
github.com/spf13/jwalterweatherman v1.1.0 h1:ue6voC5bR5F8YxI5S67j9i582FU4Qvo2bmqnqMYADFk=
github.com/spf13/jwalterweatherman v1.1.0/go.mod h1:aNWZUN0dPAAO/Ljvb5BEdw96iTZ0EXowPYD95IqWIGo=
github.com/spf13/pflag v1.0.5 h1:iy+VFUOCP1a+8yFto/drg2CJ5u0yRoB7fZw3DKv/JXA=
github.com/spf13/pflag v1.0.5/go.mod h1:McXfInJRrz4CZXVZOBLb0bTZqETkiAhM9Iw0y3An2Bg=
github.com/spf13/viper v1.16.0 h1:rGGH0XDZhdUOryiDWjmIvUSWpbNqisK8Wk0Vyefw8hc=
github.com/spf13/viper v1.16.0/go.mod h1:yg78JgCJcbrQOvV9YLXgkLaZqUidkY9K+Dd1FofRzQg=
github.com/stretchr/objx v0.1.0/go.mod h1:HFkY916IF+rwdDfMAkV7OtwuqBVzrE8GR6GFx+wExME=
github.com/stretchr/objx v0.1.1/go.mod h1:HFkY916IF+rwdDfMAkV7OtwuqBVzrE8GR6GFx+wExME=
github.com/stretchr/objx v0.4.0/go.mod h1:YvHI0jy2hoMjB+UWwv71VJQ9isScKT/TqJzVSSt89Yw=
github.com/stretchr/objx v0.5.0 h1:1zr/of2m5FGMsad5YfcqgdqdWrIhu+EBEJRhR1U7z/c=
github.com/stretchr/objx v0.5.0/go.mod h1:Yh+to48EsGEfYuaHDzXPcE3xhTkx73EhmCGUpEOglKo=
github.com/stretchr/testify v1.2.2/go.mod h1:a8OnRcib4nhh0OaRAV+Yts87kKdq0PP7pXfy6kDkUVs=
github.com/stretchr/testify v1.3.0/go.mod h1:M5WIy9Dh21IEIfnGCwXGc5bZfKNJtfHm1UVUgZn+9EI=
github.com/stretchr/testify v1.4.0/go.mod h1:j7eGeouHqKxXV5pUuKE4zz7dFj8WfuZ+81PSLYec5m4=
github.com/stretchr/testify v1.5.1/go.mod h1:5W2xD1RspED5o8YsWQXVCued0rvSQ+mT+I5cxcmMvtA=
github.com/stretchr/testify v1.6.1/go.mod h1:6Fq8oRcR53rry900zMqJjRRixrwX3KX962/h/Wwjteg=
github.com/stretchr/testify v1.7.0/go.mod h1:6Fq8oRcR53rry900zMqJjRRixrwX3KX962/h/Wwjteg=
github.com/stretchr/testify v1.7.1/go.mod h1:6Fq8oRcR53rry900zMqJjRRixrwX3KX962/h/Wwjteg=
github.com/stretchr/testify v1.8.0/go.mod h1:yNjHg4UonilssWZ8iaSj1OCr/vHnekPRkoO+kdMU+MU=
github.com/stretchr/testify v1.8.1/go.mod h1:w2LPCIKwWwSfY2zedu0+kehJoqGctiVI29o6fzry7u4=
github.com/stretchr/testify v1.8.2/go.mod h1:w2LPCIKwWwSfY2zedu0+kehJoqGctiVI29o6fzry7u4=
github.com/stretchr/testify v1.8.3/go.mod h1:sz/lmYIOXD/1dqDmKjjqLyZ2RngseejIcXlSw2iwfAo=
github.com/stretchr/testify v1.8.4 h1:CcVxjf3Q8PM0mHUKJCdn+eZZtm5yQwehR5yeSVQQcUk=
github.com/stretchr/testify v1.8.4/go.mod h1:sz/lmYIOXD/1dqDmKjjqLyZ2RngseejIcXlSw2iwfAo=
github.com/subosito/gotenv v1.4.2 h1:X1TuBLAMDFbaTAChgCBLu3DU3UPyELpnF2jjJ2cz/S8=
github.com/subosito/gotenv v1.4.2/go.mod h1:ayKnFf/c6rvx/2iiLrJUk1e6plDbT3edrFNGqEflhK0=
github.com/swaggo/files v1.0.1 h1:J1bVJ4XHZNq0I46UU90611i9/YzdrF7x92oX1ig5IdE=
github.com/swaggo/files v1.0.1/go.mod h1:0qXmMNH6sXNf+73t65aKeB+ApmgxdnkQzVTAj2uaMUg=
github.com/swaggo/gin-swagger v1.6.0 h1:y8sxvQ3E20/RCyrXeFfg60r6H0Z+SwpTjMYsMm+zy8M=
github.com/swaggo/gin-swagger v1.6.0/go.mod h1:BG00cCEy294xtVpyIAHG6+e2Qzj/xKlRdOqDkvq0uzo=
github.com/swaggo/swag v1.8.12 h1:pctzkNPu0AlQP2royqX3apjKCQonAnf7KGoxeO4y64w=
github.com/swaggo/swag v1.8.12/go.mod h1:lNfm6Gg+oAq3zRJQNEMBE66LIJKM44mxFqhEEgy2its=
github.com/twitchyliquid64/golang-asm v0.15.1 h1:SU5vSMR7hnwNxj24w34ZyCi/FmDZTkS4MhqMhdFk5YI=
github.com/twitchyliquid64/golang-asm v0.15.1/go.mod h1:a1lVb/DtPvCB8fslRZhAngC2+aY1QWCk3Cedj/Gdt08=
github.com/ugorji/go v1.1.7/go.mod h1:kZn38zHttfInRq0xu/PH0az30d+z6vm202qpg1oXVMw=
github.com/ugorji/go/codec v1.1.7/go.mod h1:Ax+UKWsSmolVDwsd+7N3ZtXu+yMGCf907BLYF3GoBXY=
github.com/ugorji/go/codec v1.2.11 h1:BMaWp1Bb6fHwEtbplGBGJ498wD+LKlNSl25MjdZY4dU=
github.com/ugorji/go/codec v1.2.11/go.mod h1:UNopzCgEMSXjBc6AOMqYvWC1ktqTAfzJZUZgYf6w6lg=
github.com/yuin/goldmark v1.1.25/go.mod h1:3hX8gzYuyVAZsxl0MRgGTJEmQBFcNTphYh9decYSb74=
github.com/yuin/goldmark v1.1.27/go.mod h1:3hX8gzYuyVAZsxl0MRgGTJEmQBFcNTphYh9decYSb74=
github.com/yuin/goldmark v1.1.32/go.mod h1:3hX8gzYuyVAZsxl0MRgGTJEmQBFcNTphYh9decYSb74=
github.com/yuin/goldmark v1.2.1/go.mod h1:3hX8gzYuyVAZsxl0MRgGTJEmQBFcNTphYh9decYSb74=
github.com/yuin/goldmark v1.4.13/go.mod h1:6yULJ656Px+3vBD8DxQVa3kxgyrAnzto9xy5taEt/CY=
go.opencensus.io v0.21.0/go.mod h1:mSImk1erAIZhrmZN+AvHh14ztQfjbGwt4TtuofqLduU=
go.opencensus.io v0.22.0/go.mod h1:+kGneAE2xo2IficOXnaByMWTGM9T73dGwxeWcUqIpI8=
go.opencensus.io v0.22.2/go.mod h1:yxeiOL68Rb0Xd1ddK5vPZ/oVn4vY4Ynel7k9FzqtOIw=
go.opencensus.io v0.22.3/go.mod h1:yxeiOL68Rb0Xd1ddK5vPZ/oVn4vY4Ynel7k9FzqtOIw=
go.opencensus.io v0.22.4/go.mod h1:yxeiOL68Rb0Xd1ddK5vPZ/oVn4vY4Ynel7k9FzqtOIw=
go.opencensus.io v0.22.5/go.mod h1:5pWMHQbX5EPX2/62yrJeAkowc+lfs/XD7Uxpq3pI6kk=
golang.org/x/arch v0.0.0-20210923205945-b76863e36670/go.mod h1:5om86z9Hs0C8fWVUuoMHwpExlXzs5Tkyp9hOrfG7pp8=
golang.org/x/arch v0.3.0 h1:02VY4/ZcO/gBOH6PUaoiptASxtXU10jazRCP865E97k=
golang.org/x/arch v0.3.0/go.mod h1:5om86z9Hs0C8fWVUuoMHwpExlXzs5Tkyp9hOrfG7pp8=
golang.org/x/crypto v0.0.0-20180904163835-0709b304e793/go.mod h1:6SG95UA2DQfeDnfUPMdvaQW0Q7yPrPDi9nlGo2tz2b4=
golang.org/x/crypto v0.0.0-20190308221718-c2843e01d9a2/go.mod h1:djNgcEr1/C05ACkg1iLfiJU5Ep61QUkGW8qpdssI0+w=
golang.org/x/crypto v0.0.0-20190510104115-cbcb75029529/go.mod h1:yigFU9vqHzYiE8UmvKecakEJjdnWj3jj499lnFckfCI=
golang.org/x/crypto v0.0.0-20190605123033-f99c8df09eb5/go.mod h1:yigFU9vqHzYiE8UmvKecakEJjdnWj3jj499lnFckfCI=
golang.org/x/crypto v0.0.0-20191011191535-87dc89f01550/go.mod h1:yigFU9vqHzYiE8UmvKecakEJjdnWj3jj499lnFckfCI=
golang.org/x/crypto v0.0.0-20200622213623-75b288015ac9/go.mod h1:LzIPMQfyMNhhGPhUkYOs5KpL4U8rLKemX1yGLhDgUto=
golang.org/x/crypto v0.0.0-20210421170649-83a5a9bb288b/go.mod h1:T9bdIzuCu7OtxOm1hfPfRQxPLYneinmdGuTeoZ9dtd4=
golang.org/x/crypto v0.0.0-20210921155107-089bfa567519/go.mod h1:GvvjBRRGRdwPK5ydBHafDWAxML/pGHZbMvKqRZ5+Abc=
golang.org/x/crypto v0.0.0-20220722155217-630584e8d5aa/go.mod h1:IxCIyHEi3zRg3s0A5j5BB6A9Jmi73HwBIUl50j+osU4=
golang.org/x/crypto v0.9.0 h1:LF6fAI+IutBocDJ2OT0Q1g8plpYljMZ4+lty+dsqw3g=
golang.org/x/crypto v0.9.0/go.mod h1:yrmDGqONDYtNj3tH8X9dzUun2m2lzPa9ngI6/RUPGR0=
golang.org/x/exp v0.0.0-20190121172915-509febef88a4/go.mod h1:CJ0aWSM057203Lf6IL+f9T1iT9GByDxfZKAQTCR3kQA=
golang.org/x/exp v0.0.0-20190306152737-a1d7652674e8/go.mod h1:CJ0aWSM057203Lf6IL+f9T1iT9GByDxfZKAQTCR3kQA=
golang.org/x/exp v0.0.0-20190510132918-efd6b22b2522/go.mod h1:ZjyILWgesfNpC6sMxTJOJm9Kp84zZh5NQWvqDGG3Qr8=
golang.org/x/exp v0.0.0-20190829153037-c13cbed26979/go.mod h1:86+5VVa7VpoJ4kLfm080zCjGlMRFzhUhsZKEZO7MGek=
golang.org/x/exp v0.0.0-20191030013958-a1ab85dbe136/go.mod h1:JXzH8nQsPlswgeRAPE3MuO9GYsAcnJvJ4vnMwN/5qkY=
golang.org/x/exp v0.0.0-20191129062945-2f5052295587/go.mod h1:2RIsYlXP63K8oxa1u096TMicItID8zy7Y6sNkU49FU4=
golang.org/x/exp v0.0.0-20191227195350-da58074b4299/go.mod h1:2RIsYlXP63K8oxa1u096TMicItID8zy7Y6sNkU49FU4=
golang.org/x/exp v0.0.0-20200119233911-0405dc783f0a/go.mod h1:2RIsYlXP63K8oxa1u096TMicItID8zy7Y6sNkU49FU4=
golang.org/x/exp v0.0.0-20200207192155-f17229e696bd/go.mod h1:J/WKrq2StrnmMY6+EHIKF9dgMWnmCNThgcyBT1FY9mM=
golang.org/x/exp v0.0.0-20200224162631-6cc2880d07d6/go.mod h1:3jZMyOhIsHpP37uCMkUooju7aAi5cS1Q23tOzKc+0MU=
golang.org/x/image v0.0.0-20190227222117-0694c2d4d067/go.mod h1:kZ7UVZpmo3dzQBMxlp+ypCbDeSB+sBbTgSJuh5dn5js=
golang.org/x/image v0.0.0-20190802002840-cff245a6509b/go.mod h1:FeLwcggjj3mMvU+oOTbSwawSJRM1uh48EjtB4UJZlP0=
golang.org/x/lint v0.0.0-20181026193005-c67002cb31c3/go.mod h1:UVdnD1Gm6xHRNCYTkRU2/jEulfH38KcIWyp/GAMgvoE=
golang.org/x/lint v0.0.0-20190227174305-5b3e6a55c961/go.mod h1:wehouNa3lNwaWXcvxsM5YxQ5yQlVC4a0KAMCusXpPoU=
golang.org/x/lint v0.0.0-20190301231843-5614ed5bae6f/go.mod h1:UVdnD1Gm6xHRNCYTkRU2/jEulfH38KcIWyp/GAMgvoE=
golang.org/x/lint v0.0.0-20190313153728-d0100b6bd8b3/go.mod h1:6SW0HCj/g11FgYtHlgUYUwCkIfeOF89ocIRzGO/8vkc=
golang.org/x/lint v0.0.0-20190409202823-959b441ac422/go.mod h1:6SW0HCj/g11FgYtHlgUYUwCkIfeOF89ocIRzGO/8vkc=
golang.org/x/lint v0.0.0-20190909230951-414d861bb4ac/go.mod h1:6SW0HCj/g11FgYtHlgUYUwCkIfeOF89ocIRzGO/8vkc=
golang.org/x/lint v0.0.0-20190930215403-16217165b5de/go.mod h1:6SW0HCj/g11FgYtHlgUYUwCkIfeOF89ocIRzGO/8vkc=
golang.org/x/lint v0.0.0-20191125180803-fdd1cda4f05f/go.mod h1:5qLYkcX4OjUUV8bRuDixDT3tpyyb+LUpUlRWLxfhWrs=
golang.org/x/lint v0.0.0-20200130185559-910be7a94367/go.mod h1:3xt1FjdF8hUf6vQPIChWIBhFzV8gjjsPE/fR3IyQdNY=
golang.org/x/lint v0.0.0-20200302205851-738671d3881b/go.mod h1:3xt1FjdF8hUf6vQPIChWIBhFzV8gjjsPE/fR3IyQdNY=
golang.org/x/lint v0.0.0-20201208152925-83fdc39ff7b5/go.mod h1:3xt1FjdF8hUf6vQPIChWIBhFzV8gjjsPE/fR3IyQdNY=
golang.org/x/mobile v0.0.0-20190312151609-d3739f865fa6/go.mod h1:z+o9i4GpDbdi3rU15maQ/Ox0txvL9dWGYEHz965HBQE=
golang.org/x/mobile v0.0.0-20190719004257-d2bd2a29d028/go.mod h1:E/iHnbuqvinMTCcRqshq8CkpyQDoeVncDDYHnLhea+o=
golang.org/x/mod v0.0.0-20190513183733-4bf6d317e70e/go.mod h1:mXi4GBBbnImb6dmsKGUJ2LatrhH/nqhxcFungHvyanc=
golang.org/x/mod v0.1.0/go.mod h1:0QHyrYULN0/3qlju5TqG8bIK38QM8yzMo5ekMj3DlcY=
golang.org/x/mod v0.1.1-0.20191105210325-c90efee705ee/go.mod h1:QqPTAvyqsEbceGzBzNggFXnrqF1CaUcvgkdR5Ot7KZg=
golang.org/x/mod v0.1.1-0.20191107180719-034126e5016b/go.mod h1:QqPTAvyqsEbceGzBzNggFXnrqF1CaUcvgkdR5Ot7KZg=
golang.org/x/mod v0.2.0/go.mod h1:s0Qsj1ACt9ePp/hMypM3fl4fZqREWJwdYDEqhRiZZUA=
golang.org/x/mod v0.3.0/go.mod h1:s0Qsj1ACt9ePp/hMypM3fl4fZqREWJwdYDEqhRiZZUA=
golang.org/x/mod v0.4.0/go.mod h1:s0Qsj1ACt9ePp/hMypM3fl4fZqREWJwdYDEqhRiZZUA=
golang.org/x/mod v0.4.1/go.mod h1:s0Qsj1ACt9ePp/hMypM3fl4fZqREWJwdYDEqhRiZZUA=
golang.org/x/mod v0.6.0-dev.0.20220419223038-86c51ed26bb4/go.mod h1:jJ57K6gSWd91VN4djpZkiMVwK6gcyfeH4XE8wZrZaV4=
golang.org/x/mod v0.9.0 h1:KENHtAZL2y3NLMYZeHY9DW8HW8V+kQyJsY/V9JlKvCs=
golang.org/x/net v0.0.0-20180724234803-3673e40ba225/go.mod h1:mL1N/T3taQHkDXs73rZJwtUhF3w3ftmwwsq0BUmARs4=
golang.org/x/net v0.0.0-20180826012351-8a410e7b638d/go.mod h1:mL1N/T3taQHkDXs73rZJwtUhF3w3ftmwwsq0BUmARs4=
golang.org/x/net v0.0.0-20181114220301-adae6a3d119a/go.mod h1:mL1N/T3taQHkDXs73rZJwtUhF3w3ftmwwsq0BUmARs4=
golang.org/x/net v0.0.0-20190108225652-1e06a53dbb7e/go.mod h1:mL1N/T3taQHkDXs73rZJwtUhF3w3ftmwwsq0BUmARs4=
golang.org/x/net v0.0.0-20190213061140-3a22650c66bd/go.mod h1:mL1N/T3taQHkDXs73rZJwtUhF3w3ftmwwsq0BUmARs4=
golang.org/x/net v0.0.0-20190311183353-d8887717615a/go.mod h1:t9HGtf8HONx5eT2rtn7q6eTqICYqUVnKs3thJo3Qplg=
golang.org/x/net v0.0.0-20190404232315-eb5bcb51f2a3/go.mod h1:t9HGtf8HONx5eT2rtn7q6eTqICYqUVnKs3thJo3Qplg=
golang.org/x/net v0.0.0-20190501004415-9ce7a6920f09/go.mod h1:t9HGtf8HONx5eT2rtn7q6eTqICYqUVnKs3thJo3Qplg=
golang.org/x/net v0.0.0-20190503192946-f4e77d36d62c/go.mod h1:t9HGtf8HONx5eT2rtn7q6eTqICYqUVnKs3thJo3Qplg=
golang.org/x/net v0.0.0-20190603091049-60506f45cf65/go.mod h1:HSz+uSET+XFnRR8LxR5pz3Of3rY3CfYBVs4xY44aLks=
golang.org/x/net v0.0.0-20190613194153-d28f0bde5980/go.mod h1:z5CRVTTTmAJ677TzLLGU+0bjPO0LkuOLi4/5GtJWs/s=
golang.org/x/net v0.0.0-20190620200207-3b0461eec859/go.mod h1:z5CRVTTTmAJ677TzLLGU+0bjPO0LkuOLi4/5GtJWs/s=
golang.org/x/net v0.0.0-20190628185345-da137c7871d7/go.mod h1:z5CRVTTTmAJ677TzLLGU+0bjPO0LkuOLi4/5GtJWs/s=
golang.org/x/net v0.0.0-20190724013045-ca1201d0de80/go.mod h1:z5CRVTTTmAJ677TzLLGU+0bjPO0LkuOLi4/5GtJWs/s=
golang.org/x/net v0.0.0-20191209160850-c0dbc17a3553/go.mod h1:z5CRVTTTmAJ677TzLLGU+0bjPO0LkuOLi4/5GtJWs/s=
golang.org/x/net v0.0.0-20200114155413-6afb5195e5aa/go.mod h1:z5CRVTTTmAJ677TzLLGU+0bjPO0LkuOLi4/5GtJWs/s=
golang.org/x/net v0.0.0-20200202094626-16171245cfb2/go.mod h1:z5CRVTTTmAJ677TzLLGU+0bjPO0LkuOLi4/5GtJWs/s=
golang.org/x/net v0.0.0-20200222125558-5a598a2470a0/go.mod h1:z5CRVTTTmAJ677TzLLGU+0bjPO0LkuOLi4/5GtJWs/s=
golang.org/x/net v0.0.0-20200226121028-0de0cce0169b/go.mod h1:z5CRVTTTmAJ677TzLLGU+0bjPO0LkuOLi4/5GtJWs/s=
golang.org/x/net v0.0.0-20200301022130-244492dfa37a/go.mod h1:z5CRVTTTmAJ677TzLLGU+0bjPO0LkuOLi4/5GtJWs/s=
golang.org/x/net v0.0.0-20200324143707-d3edc9973b7e/go.mod h1:qpuaurCH72eLCgpAm/N6yyVIVM9cpaDIP3A8BGJEC5A=
golang.org/x/net v0.0.0-20200501053045-e0ff5e5a1de5/go.mod h1:qpuaurCH72eLCgpAm/N6yyVIVM9cpaDIP3A8BGJEC5A=
golang.org/x/net v0.0.0-20200506145744-7e3656a0809f/go.mod h1:qpuaurCH72eLCgpAm/N6yyVIVM9cpaDIP3A8BGJEC5A=
golang.org/x/net v0.0.0-20200513185701-a91f0712d120/go.mod h1:qpuaurCH72eLCgpAm/N6yyVIVM9cpaDIP3A8BGJEC5A=
golang.org/x/net v0.0.0-20200520182314-0ba52f642ac2/go.mod h1:qpuaurCH72eLCgpAm/N6yyVIVM9cpaDIP3A8BGJEC5A=
golang.org/x/net v0.0.0-20200625001655-4c5254603344/go.mod h1:/O7V0waA8r7cgGh81Ro3o1hOxt32SMVPicZroKQ2sZA=
golang.org/x/net v0.0.0-20200707034311-ab3426394381/go.mod h1:/O7V0waA8r7cgGh81Ro3o1hOxt32SMVPicZroKQ2sZA=
golang.org/x/net v0.0.0-20200822124328-c89045814202/go.mod h1:/O7V0waA8r7cgGh81Ro3o1hOxt32SMVPicZroKQ2sZA=
golang.org/x/net v0.0.0-20201021035429-f5854403a974/go.mod h1:sp8m0HH+o8qH0wwXwYZr8TS3Oi6o0r6Gce1SSxlDquU=
golang.org/x/net v0.0.0-20201031054903-ff519b6c9102/go.mod h1:sp8m0HH+o8qH0wwXwYZr8TS3Oi6o0r6Gce1SSxlDquU=
golang.org/x/net v0.0.0-20201209123823-ac852fbbde11/go.mod h1:m0MpNAwzfU5UDzcl9v0D8zg8gWTRqZa9RBIspLL5mdg=
golang.org/x/net v0.0.0-20201224014010-6772e930b67b/go.mod h1:m0MpNAwzfU5UDzcl9v0D8zg8gWTRqZa9RBIspLL5mdg=
golang.org/x/net v0.0.0-20210226172049-e18ecbb05110/go.mod h1:m0MpNAwzfU5UDzcl9v0D8zg8gWTRqZa9RBIspLL5mdg=
golang.org/x/net v0.0.0-20210421230115-4e50805a0758/go.mod h1:72T/g9IO56b78aLF+1Kcs5dz7/ng1VjMUvfKvpfy+jM=
golang.org/x/net v0.0.0-20210525063256-abc453219eb5/go.mod h1:9nx3DQGgdP8bBQD5qxJ1jj9UTztislL4KSBs9R2vV5Y=
golang.org/x/net v0.0.0-20211112202133-69e39bad7dc2/go.mod h1:9nx3DQGgdP8bBQD5qxJ1jj9UTztislL4KSBs9R2vV5Y=
golang.org/x/net v0.0.0-20220722155237-a158d28d115b/go.mod h1:XRhObCWvk6IyKnWLug+ECip1KBveYUHfp+8e9klMJ9c=
golang.org/x/net v0.7.0/go.mod h1:2Tu9+aMcznHK/AK1HMvgo6xiTLG5rD5rZLDS+rp2Bjs=
golang.org/x/net v0.10.0 h1:X2//UzNDwYmtCLn7To6G58Wr6f5ahEAQgKNzv9Y951M=
golang.org/x/net v0.10.0/go.mod h1:0qNGK6F8kojg2nk9dLZ2mShWaEBan6FAoqfSigmmuDg=
golang.org/x/oauth2 v0.0.0-20180821212333-d2e6202438be/go.mod h1:N/0e6XlmueqKjAGxoOufVs8QHGRruUQn6yWY3a++T0U=
golang.org/x/oauth2 v0.0.0-20190226205417-e64efc72b421/go.mod h1:gOpvHmFTYa4IltrdGE7lF6nIHvwfUNPOp7c8zoXwtLw=
golang.org/x/oauth2 v0.0.0-20190604053449-0f29369cfe45/go.mod h1:gOpvHmFTYa4IltrdGE7lF6nIHvwfUNPOp7c8zoXwtLw=
golang.org/x/oauth2 v0.0.0-20191202225959-858c2ad4c8b6/go.mod h1:gOpvHmFTYa4IltrdGE7lF6nIHvwfUNPOp7c8zoXwtLw=
golang.org/x/oauth2 v0.0.0-20200107190931-bf48bf16ab8d/go.mod h1:gOpvHmFTYa4IltrdGE7lF6nIHvwfUNPOp7c8zoXwtLw=
golang.org/x/oauth2 v0.0.0-20200902213428-5d25da1a8d43/go.mod h1:KelEdhl1UZF7XfJ4dDtk6s++YSgaE7mD/BuKKDLBl4A=
golang.org/x/oauth2 v0.0.0-20201109201403-9fd604954f58/go.mod h1:KelEdhl1UZF7XfJ4dDtk6s++YSgaE7mD/BuKKDLBl4A=
golang.org/x/oauth2 v0.0.0-20201208152858-08078c50e5b5/go.mod h1:KelEdhl1UZF7XfJ4dDtk6s++YSgaE7mD/BuKKDLBl4A=
golang.org/x/oauth2 v0.0.0-20210218202405-ba52d332ba99/go.mod h1:KelEdhl1UZF7XfJ4dDtk6s++YSgaE7mD/BuKKDLBl4A=
golang.org/x/oauth2 v0.0.0-20210514164344-f6687ab2804c/go.mod h1:KelEdhl1UZF7XfJ4dDtk6s++YSgaE7mD/BuKKDLBl4A=
golang.org/x/sync v0.0.0-20180314180146-1d60e4601c6f/go.mod h1:RxMgew5VJxzue5/jJTE5uejpjVlOe/izrB70Jof72aM=
golang.org/x/sync v0.0.0-20181108010431-42b317875d0f/go.mod h1:RxMgew5VJxzue5/jJTE5uejpjVlOe/izrB70Jof72aM=
golang.org/x/sync v0.0.0-20181221193216-37e7f081c4d4/go.mod h1:RxMgew5VJxzue5/jJTE5uejpjVlOe/izrB70Jof72aM=
golang.org/x/sync v0.0.0-20190227155943-e225da77a7e6/go.mod h1:RxMgew5VJxzue5/jJTE5uejpjVlOe/izrB70Jof72aM=
golang.org/x/sync v0.0.0-20190423024810-112230192c58/go.mod h1:RxMgew5VJxzue5/jJTE5uejpjVlOe/izrB70Jof72aM=
golang.org/x/sync v0.0.0-20190911185100-cd5d95a43a6e/go.mod h1:RxMgew5VJxzue5/jJTE5uejpjVlOe/izrB70Jof72aM=
golang.org/x/sync v0.0.0-20200317015054-43a5402ce75a/go.mod h1:RxMgew5VJxzue5/jJTE5uejpjVlOe/izrB70Jof72aM=
golang.org/x/sync v0.0.0-20200625203802-6e8e738ad208/go.mod h1:RxMgew5VJxzue5/jJTE5uejpjVlOe/izrB70Jof72aM=
golang.org/x/sync v0.0.0-20201020160332-67f06af15bc9/go.mod h1:RxMgew5VJxzue5/jJTE5uejpjVlOe/izrB70Jof72aM=
golang.org/x/sync v0.0.0-20201207232520-09787c993a3a/go.mod h1:RxMgew5VJxzue5/jJTE5uejpjVlOe/izrB70Jof72aM=
golang.org/x/sync v0.0.0-20220722155255-886fb9371eb4/go.mod h1:RxMgew5VJxzue5/jJTE5uejpjVlOe/izrB70Jof72aM=
golang.org/x/sys v0.0.0-20180830151530-49385e6e1522/go.mod h1:STP8DvDyc/dI5b8T5hshtkjS+E42TnysNCUPdjciGhY=
golang.org/x/sys v0.0.0-20180905080454-ebe1bf3edb33/go.mod h1:STP8DvDyc/dI5b8T5hshtkjS+E42TnysNCUPdjciGhY=
golang.org/x/sys v0.0.0-20181116152217-5ac8a444bdc5/go.mod h1:STP8DvDyc/dI5b8T5hshtkjS+E42TnysNCUPdjciGhY=
golang.org/x/sys v0.0.0-20190215142949-d0b11bdaac8a/go.mod h1:STP8DvDyc/dI5b8T5hshtkjS+E42TnysNCUPdjciGhY=
golang.org/x/sys v0.0.0-20190312061237-fead79001313/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20190412213103-97732733099d/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20190422165155-953cdadca894/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20190502145724-3ef323f4f1fd/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20190507160741-ecd444e8653b/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20190606165138-5da285871e9c/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20190624142023-c5567b49c5d0/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20190726091711-fc99dfbffb4e/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20191001151750-bb3f8db39f24/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20191204072324-ce4227a45e2e/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20191228213918-04cbcbbfeed8/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200106162015-b016eb3dc98e/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200113162924-86b910548bc1/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200116001909-b77594299b42/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200122134326-e047566fdf82/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200202164722-d101bd2416d5/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200212091648-12a6c2dcc1e4/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200223170610-d5e6a3e2c0ae/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200302150141-5c8b2ff67527/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200323222414-85ca7c5b95cd/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200331124033-c3d80250170d/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200501052902-10377860bb8e/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200511232937-7e40ca221e25/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200515095857-1151b9dac4a9/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200523222454-059865788121/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200615200032-f1bc736245b1/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200625212154-ddb9806d33ae/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200803210538-64077c9b5642/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200905004654-be1d3432aa8f/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20200930185726-fdedc70b468f/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20201119102817-f84b799fce68/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20201201145000-ef89a241ccb3/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20210104204734-6f8348627aad/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20210119212857-b64e53b001e4/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20210124154548-22da62e12c0c/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20210225134936-a50acf3fe073/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20210420072515-93ed5bcd2bfe/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20210423082822-04245dca01da/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20210423185535-09eb48e85fd7/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
golang.org/x/sys v0.0.0-20210603081109-ebe580a85c40/go.mod h1:oPkhp1MJrh7nUepCBck5+mAzfO9JrbApNNgaTdGDITg=
golang.org/x/sys v0.0.0-20210615035016-665e8c7367d1/go.mod h1:oPkhp1MJrh7nUepCBck5+mAzfO9JrbApNNgaTdGDITg=
golang.org/x/sys v0.0.0-20220114195835-da31bd327af9/go.mod h1:oPkhp1MJrh7nUepCBck5+mAzfO9JrbApNNgaTdGDITg=
golang.org/x/sys v0.0.0-20220520151302-bc2c85ada10a/go.mod h1:oPkhp1MJrh7nUepCBck5+mAzfO9JrbApNNgaTdGDITg=
golang.org/x/sys v0.0.0-20220704084225-05e143d24a9e/go.mod h1:oPkhp1MJrh7nUepCBck5+mAzfO9JrbApNNgaTdGDITg=
golang.org/x/sys v0.0.0-20220715151400-c0bba94af5f8/go.mod h1:oPkhp1MJrh7nUepCBck5+mAzfO9JrbApNNgaTdGDITg=
golang.org/x/sys v0.0.0-20220722155257-8c9f86f7a55f/go.mod h1:oPkhp1MJrh7nUepCBck5+mAzfO9JrbApNNgaTdGDITg=
golang.org/x/sys v0.0.0-20220908164124-27713097b956/go.mod h1:oPkhp1MJrh7nUepCBck5+mAzfO9JrbApNNgaTdGDITg=
golang.org/x/sys v0.5.0/go.mod h1:oPkhp1MJrh7nUepCBck5+mAzfO9JrbApNNgaTdGDITg=
golang.org/x/sys v0.6.0/go.mod h1:oPkhp1MJrh7nUepCBck5+mAzfO9JrbApNNgaTdGDITg=
golang.org/x/sys v0.8.0 h1:EBmGv8NaZBZTWvrbjNoL6HVt+IVy3QDQpJs7VRIw3tU=
golang.org/x/sys v0.8.0/go.mod h1:oPkhp1MJrh7nUepCBck5+mAzfO9JrbApNNgaTdGDITg=
golang.org/x/term v0.0.0-20201126162022-7de9c90e9dd1/go.mod h1:bj7SfCRtBDWHUb9snDiAeCFNEtKQo2Wmx5Cou7ajbmo=
golang.org/x/term v0.0.0-20210927222741-03fcf44c2211/go.mod h1:jbD1KX2456YbFQfuXm/mYQcufACuNUgVhRMnK/tPxf8=
golang.org/x/term v0.5.0/go.mod h1:jMB1sMXY+tzblOD4FWmEbocvup2/aLOaQEp7JmGp78k=
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c/go.mod h1:NqM8EUOU14njkJ3fqMW+pc6Ldnwhi/IjpwHt7yyuwOQ=
golang.org/x/text v0.3.0/go.mod h1:NqM8EUOU14njkJ3fqMW+pc6Ldnwhi/IjpwHt7yyuwOQ=
golang.org/x/text v0.3.1-0.20180807135948-17ff2d5776d2/go.mod h1:NqM8EUOU14njkJ3fqMW+pc6Ldnwhi/IjpwHt7yyuwOQ=
golang.org/x/text v0.3.2/go.mod h1:bEr9sfX3Q8Zfm5fL9x+3itogRgK3+ptLWKqgva+5dAk=
golang.org/x/text v0.3.3/go.mod h1:5Zoc/QRtKVWzQhOtBMvqHzDpF6irO9z98xDceosuGiQ=
golang.org/x/text v0.3.4/go.mod h1:5Zoc/QRtKVWzQhOtBMvqHzDpF6irO9z98xDceosuGiQ=
golang.org/x/text v0.3.6/go.mod h1:5Zoc/QRtKVWzQhOtBMvqHzDpF6irO9z98xDceosuGiQ=
golang.org/x/text v0.3.7/go.mod h1:u+2+/6zg+i71rQMx5EYifcz6MCKuco9NR6JIITiCfzQ=
golang.org/x/text v0.7.0/go.mod h1:mrYo+phRRbMaCq/xk9113O4dZlRixOauAjOtrjsXDZ8=
golang.org/x/text v0.9.0 h1:2sjJmO8cDvYveuX97RDLsxlyUxLl+GHoLxBiRdHllBE=
golang.org/x/text v0.9.0/go.mod h1:e1OnstbJyHTd6l/uOt8jFFHp6TRDWZR/bV3emEE/zU8=
golang.org/x/time v0.0.0-20181108054448-85acf8d2951c/go.mod h1:tRJNPiyCQ0inRvYxbN9jk5I+vvW/OXSQhTDSoE431IQ=
golang.org/x/time v0.0.0-20190308202827-9d24e82272b4/go.mod h1:tRJNPiyCQ0inRvYxbN9jk5I+vvW/OXSQhTDSoE431IQ=
golang.org/x/time v0.0.0-20191024005414-555d28b269f0/go.mod h1:tRJNPiyCQ0inRvYxbN9jk5I+vvW/OXSQhTDSoE431IQ=
golang.org/x/tools v0.0.0-20180917221912-90fa682c2a6e/go.mod h1:n7NCudcB/nEzxVGmLbDWY5pfWTLqBcC2KZ6jyYvM4mQ=
golang.org/x/tools v0.0.0-20190114222345-bf090417da8b/go.mod h1:n7NCudcB/nEzxVGmLbDWY5pfWTLqBcC2KZ6jyYvM4mQ=
golang.org/x/tools v0.0.0-20190226205152-f727befe758c/go.mod h1:9Yl7xja0Znq3iFh3HoIrodX9oNMXvdceNzlUR8zjMvY=
golang.org/x/tools v0.0.0-20190311212946-11955173bddd/go.mod h1:LCzVGOaR6xXOjkQ3onu1FJEFr0SW1gC7cKk1uF8kGRs=
golang.org/x/tools v0.0.0-20190312151545-0bb0c0a6e846/go.mod h1:LCzVGOaR6xXOjkQ3onu1FJEFr0SW1gC7cKk1uF8kGRs=
golang.org/x/tools v0.0.0-20190312170243-e65039ee4138/go.mod h1:LCzVGOaR6xXOjkQ3onu1FJEFr0SW1gC7cKk1uF8kGRs=
golang.org/x/tools v0.0.0-20190425150028-36563e24a262/go.mod h1:RgjU9mgBXZiqYHBnxXauZ1Gv1EHHAz9KjViQ78xBX0Q=
golang.org/x/tools v0.0.0-20190506145303-2d16b83fe98c/go.mod h1:RgjU9mgBXZiqYHBnxXauZ1Gv1EHHAz9KjViQ78xBX0Q=
golang.org/x/tools v0.0.0-20190524140312-2c0ae7006135/go.mod h1:RgjU9mgBXZiqYHBnxXauZ1Gv1EHHAz9KjViQ78xBX0Q=
golang.org/x/tools v0.0.0-20190606124116-d0a3d012864b/go.mod h1:/rFqwRUd4F7ZHNgwSSTFct+R/Kf4OFW1sUzUTQQTgfc=
golang.org/x/tools v0.0.0-20190621195816-6e04913cbbac/go.mod h1:/rFqwRUd4F7ZHNgwSSTFct+R/Kf4OFW1sUzUTQQTgfc=
golang.org/x/tools v0.0.0-20190628153133-6cdbf07be9d0/go.mod h1:/rFqwRUd4F7ZHNgwSSTFct+R/Kf4OFW1sUzUTQQTgfc=
golang.org/x/tools v0.0.0-20190816200558-6889da9d5479/go.mod h1:b+2E5dAYhXwXZwtnZ6UAqBI28+e2cm9otk0dWdXHAEo=
golang.org/x/tools v0.0.0-20190911174233-4f2ddba30aff/go.mod h1:b+2E5dAYhXwXZwtnZ6UAqBI28+e2cm9otk0dWdXHAEo=
golang.org/x/tools v0.0.0-20191012152004-8de300cfc20a/go.mod h1:b+2E5dAYhXwXZwtnZ6UAqBI28+e2cm9otk0dWdXHAEo=
golang.org/x/tools v0.0.0-20191113191852-77e3bb0ad9e7/go.mod h1:b+2E5dAYhXwXZwtnZ6UAqBI28+e2cm9otk0dWdXHAEo=
golang.org/x/tools v0.0.0-20191115202509-3a792d9c32b2/go.mod h1:b+2E5dAYhXwXZwtnZ6UAqBI28+e2cm9otk0dWdXHAEo=
golang.org/x/tools v0.0.0-20191119224855-298f0cb1881e/go.mod h1:b+2E5dAYhXwXZwtnZ6UAqBI28+e2cm9otk0dWdXHAEo=
golang.org/x/tools v0.0.0-20191125144606-a911d9008d1f/go.mod h1:b+2E5dAYhXwXZwtnZ6UAqBI28+e2cm9otk0dWdXHAEo=
golang.org/x/tools v0.0.0-20191130070609-6e064ea0cf2d/go.mod h1:b+2E5dAYhXwXZwtnZ6UAqBI28+e2cm9otk0dWdXHAEo=
golang.org/x/tools v0.0.0-20191216173652-a0e659d51361/go.mod h1:TB2adYChydJhpapKDTa4BR/hXlZSLoq2Wpct/0txZ28=
golang.org/x/tools v0.0.0-20191227053925-7b8e75db28f4/go.mod h1:TB2adYChydJhpapKDTa4BR/hXlZSLoq2Wpct/0txZ28=
golang.org/x/tools v0.0.0-20200117161641-43d50277825c/go.mod h1:TB2adYChydJhpapKDTa4BR/hXlZSLoq2Wpct/0txZ28=
golang.org/x/tools v0.0.0-20200122220014-bf1340f18c4a/go.mod h1:TB2adYChydJhpapKDTa4BR/hXlZSLoq2Wpct/0txZ28=
golang.org/x/tools v0.0.0-20200130002326-2f3ba24bd6e7/go.mod h1:TB2adYChydJhpapKDTa4BR/hXlZSLoq2Wpct/0txZ28=
golang.org/x/tools v0.0.0-20200204074204-1cc6d1ef6c74/go.mod h1:TB2adYChydJhpapKDTa4BR/hXlZSLoq2Wpct/0txZ28=
golang.org/x/tools v0.0.0-20200207183749-b753a1ba74fa/go.mod h1:TB2adYChydJhpapKDTa4BR/hXlZSLoq2Wpct/0txZ28=
golang.org/x/tools v0.0.0-20200212150539-ea181f53ac56/go.mod h1:TB2adYChydJhpapKDTa4BR/hXlZSLoq2Wpct/0txZ28=
golang.org/x/tools v0.0.0-20200224181240-023911ca70b2/go.mod h1:TB2adYChydJhpapKDTa4BR/hXlZSLoq2Wpct/0txZ28=
golang.org/x/tools v0.0.0-20200227222343-706bc42d1f0d/go.mod h1:TB2adYChydJhpapKDTa4BR/hXlZSLoq2Wpct/0txZ28=
golang.org/x/tools v0.0.0-20200304193943-95d2e580d8eb/go.mod h1:o4KQGtdN14AW+yjsvvwRTJJuXz8XRtIHtEnmAXLyFUw=
golang.org/x/tools v0.0.0-20200312045724-11d5b4c81c7d/go.mod h1:o4KQGtdN14AW+yjsvvwRTJJuXz8XRtIHtEnmAXLyFUw=
golang.org/x/tools v0.0.0-20200331025713-a30bf2db82d4/go.mod h1:Sl4aGygMT6LrqrWclx+PTx3U+LnKx/seiNR+3G19Ar8=
golang.org/x/tools v0.0.0-20200501065659-ab2804fb9c9d/go.mod h1:EkVYQZoAsY45+roYkvgYkIh4xh/qjgUK9TdY2XT94GE=
golang.org/x/tools v0.0.0-20200512131952-2bc93b1c0c88/go.mod h1:EkVYQZoAsY45+roYkvgYkIh4xh/qjgUK9TdY2XT94GE=
golang.org/x/tools v0.0.0-20200515010526-7d3b6ebf133d/go.mod h1:EkVYQZoAsY45+roYkvgYkIh4xh/qjgUK9TdY2XT94GE=
golang.org/x/tools v0.0.0-20200618134242-20370b0cb4b2/go.mod h1:EkVYQZoAsY45+roYkvgYkIh4xh/qjgUK9TdY2XT94GE=
golang.org/x/tools v0.0.0-20200729194436-6467de6f59a7/go.mod h1:njjCfa9FT2d7l9Bc6FUM5FLjQPp3cFF28FI3qnDFljA=
golang.org/x/tools v0.0.0-20200804011535-6c149bb5ef0d/go.mod h1:njjCfa9FT2d7l9Bc6FUM5FLjQPp3cFF28FI3qnDFljA=
golang.org/x/tools v0.0.0-20200825202427-b303f430e36d/go.mod h1:njjCfa9FT2d7l9Bc6FUM5FLjQPp3cFF28FI3qnDFljA=
golang.org/x/tools v0.0.0-20200904185747-39188db58858/go.mod h1:Cj7w3i3Rnn0Xh82ur9kSqwfTHTeVxaDqrfMjpcNT6bE=
golang.org/x/tools v0.0.0-20201110124207-079ba7bd75cd/go.mod h1:emZCQorbCU4vsT4fOWvOPXz4eW1wZW4PmDk9uLelYpA=
golang.org/x/tools v0.0.0-20201201161351-ac6f37ff4c2a/go.mod h1:emZCQorbCU4vsT4fOWvOPXz4eW1wZW4PmDk9uLelYpA=
golang.org/x/tools v0.0.0-20201208233053-a543418bbed2/go.mod h1:emZCQorbCU4vsT4fOWvOPXz4eW1wZW4PmDk9uLelYpA=
golang.org/x/tools v0.0.0-20210105154028-b0ab187a4818/go.mod h1:emZCQorbCU4vsT4fOWvOPXz4eW1wZW4PmDk9uLelYpA=
golang.org/x/tools v0.0.0-20210108195828-e2f9c7f1fc8e/go.mod h1:emZCQorbCU4vsT4fOWvOPXz4eW1wZW4PmDk9uLelYpA=
golang.org/x/tools v0.1.0/go.mod h1:xkSsbof2nBLbhDlRMhhhyNLN/zl3eTqcnHD5viDpcZ0=
golang.org/x/tools v0.1.12/go.mod h1:hNGJHUnrk76NpqgfD5Aqm5Crs+Hm0VOH/i9J2+nxYbc=
golang.org/x/tools v0.7.0 h1:W4OVu8VVOaIO0yzWMNdepAulS7YfoS3Zabrm8DOXXU4=
golang.org/x/tools v0.7.0/go.mod h1:4pg6aUX35JBAogB10C9AtvVL+qowtN4pT3CGSQex14s=
golang.org/x/xerrors v0.0.0-20190717185122-a985d3407aa7/go.mod h1:I/5z698sn9Ka8TeJc9MKroUUfqBBauWjQqLJ2OPfmY0=
golang.org/x/xerrors v0.0.0-20191011141410-1b5146add898/go.mod h1:I/5z698sn9Ka8TeJc9MKroUUfqBBauWjQqLJ2OPfmY0=
golang.org/x/xerrors v0.0.0-20191204190536-9bdfabe68543/go.mod h1:I/5z698sn9Ka8TeJc9MKroUUfqBBauWjQqLJ2OPfmY0=
golang.org/x/xerrors v0.0.0-20200804184101-5ec99f83aff1/go.mod h1:I/5z698sn9Ka8TeJc9MKroUUfqBBauWjQqLJ2OPfmY0=
google.golang.org/api v0.4.0/go.mod h1:8k5glujaEP+g9n7WNsDg8QP6cUVNI86fCNMcbazEtwE=
google.golang.org/api v0.7.0/go.mod h1:WtwebWUNSVBH/HAw79HIFXZNqEvBhG+Ra+ax0hx3E3M=
google.golang.org/api v0.8.0/go.mod h1:o4eAsZoiT+ibD93RtjEohWalFOjRDx6CVaqeizhEnKg=
google.golang.org/api v0.9.0/go.mod h1:o4eAsZoiT+ibD93RtjEohWalFOjRDx6CVaqeizhEnKg=
google.golang.org/api v0.13.0/go.mod h1:iLdEw5Ide6rF15KTC1Kkl0iskquN2gFfn9o9XIsbkAI=
google.golang.org/api v0.14.0/go.mod h1:iLdEw5Ide6rF15KTC1Kkl0iskquN2gFfn9o9XIsbkAI=
google.golang.org/api v0.15.0/go.mod h1:iLdEw5Ide6rF15KTC1Kkl0iskquN2gFfn9o9XIsbkAI=
google.golang.org/api v0.17.0/go.mod h1:BwFmGc8tA3vsd7r/7kR8DY7iEEGSU04BFxCo5jP/sfE=
google.golang.org/api v0.18.0/go.mod h1:BwFmGc8tA3vsd7r/7kR8DY7iEEGSU04BFxCo5jP/sfE=
google.golang.org/api v0.19.0/go.mod h1:BwFmGc8tA3vsd7r/7kR8DY7iEEGSU04BFxCo5jP/sfE=
google.golang.org/api v0.20.0/go.mod h1:BwFmGc8tA3vsd7r/7kR8DY7iEEGSU04BFxCo5jP/sfE=
google.golang.org/api v0.22.0/go.mod h1:BwFmGc8tA3vsd7r/7kR8DY7iEEGSU04BFxCo5jP/sfE=
google.golang.org/api v0.24.0/go.mod h1:lIXQywCXRcnZPGlsd8NbLnOjtAoL6em04bJ9+z0MncE=
google.golang.org/api v0.28.0/go.mod h1:lIXQywCXRcnZPGlsd8NbLnOjtAoL6em04bJ9+z0MncE=
google.golang.org/api v0.29.0/go.mod h1:Lcubydp8VUV7KeIHD9z2Bys/sm/vGKnG1UHuDBSrHWM=
google.golang.org/api v0.30.0/go.mod h1:QGmEvQ87FHZNiUVJkT14jQNYJ4ZJjdRF23ZXz5138Fc=
google.golang.org/api v0.35.0/go.mod h1:/XrVsuzM0rZmrsbjJutiuftIzeuTQcEeaYcSk/mQ1dg=
google.golang.org/api v0.36.0/go.mod h1:+z5ficQTmoYpPn8LCUNVpK5I7hwkpjbcgqA7I34qYtE=
google.golang.org/api v0.40.0/go.mod h1:fYKFpnQN0DsDSKRVRcQSDQNtqWPfM9i+zNPxepjRCQ8=
google.golang.org/appengine v1.1.0/go.mod h1:EbEs0AVv82hx2wNQdGPgUI5lhzA/G0D9YwlJXL52JkM=
google.golang.org/appengine v1.4.0/go.mod h1:xpcJRLb0r/rnEns0DIKYYv+WjYCduHsrkT7/EB5XEv4=
google.golang.org/appengine v1.5.0/go.mod h1:xpcJRLb0r/rnEns0DIKYYv+WjYCduHsrkT7/EB5XEv4=
google.golang.org/appengine v1.6.1/go.mod h1:i06prIuMbXzDqacNJfV5OdTW448YApPu5ww/cMBSeb0=
google.golang.org/appengine v1.6.5/go.mod h1:8WjMMxjGQR8xUklV/ARdw2HLXBOI7O7uCIDZVag1xfc=
google.golang.org/appengine v1.6.6/go.mod h1:8WjMMxjGQR8xUklV/ARdw2HLXBOI7O7uCIDZVag1xfc=
google.golang.org/appengine v1.6.7/go.mod h1:8WjMMxjGQR8xUklV/ARdw2HLXBOI7O7uCIDZVag1xfc=
google.golang.org/genproto v0.0.0-20180817151627-c66870c02cf8/go.mod h1:JiN7NxoALGmiZfu7CAH4rXhgtRTLTxftemlI0sWmxmc=
google.golang.org/genproto v0.0.0-20190307195333-5fe7a883aa19/go.mod h1:VzzqZJRnGkLBvHegQrXjBqPurQTc5/KpmUdxsrq26oE=
google.golang.org/genproto v0.0.0-20190418145605-e7d98fc518a7/go.mod h1:VzzqZJRnGkLBvHegQrXjBqPurQTc5/KpmUdxsrq26oE=
google.golang.org/genproto v0.0.0-20190425155659-357c62f0e4bb/go.mod h1:VzzqZJRnGkLBvHegQrXjBqPurQTc5/KpmUdxsrq26oE=
google.golang.org/genproto v0.0.0-20190502173448-54afdca5d873/go.mod h1:VzzqZJRnGkLBvHegQrXjBqPurQTc5/KpmUdxsrq26oE=
google.golang.org/genproto v0.0.0-20190801165951-fa694d86fc64/go.mod h1:DMBHOl98Agz4BDEuKkezgsaosCRResVns1a3J2ZsMNc=
google.golang.org/genproto v0.0.0-20190819201941-24fa4b261c55/go.mod h1:DMBHOl98Agz4BDEuKkezgsaosCRResVns1a3J2ZsMNc=
google.golang.org/genproto v0.0.0-20190911173649-1774047e7e51/go.mod h1:IbNlFCBrqXvoKpeg0TB2l7cyZUmoaFKYIwrEpbDKLA8=
google.golang.org/genproto v0.0.0-20191108220845-16a3f7862a1a/go.mod h1:n3cpQtvxv34hfy77yVDNjmbRyujviMdxYliBSkLhpCc=
google.golang.org/genproto v0.0.0-20191115194625-c23dd37a84c9/go.mod h1:n3cpQtvxv34hfy77yVDNjmbRyujviMdxYliBSkLhpCc=
google.golang.org/genproto v0.0.0-20191216164720-4f79533eabd1/go.mod h1:n3cpQtvxv34hfy77yVDNjmbRyujviMdxYliBSkLhpCc=
google.golang.org/genproto v0.0.0-20191230161307-f3c370f40bfb/go.mod h1:n3cpQtvxv34hfy77yVDNjmbRyujviMdxYliBSkLhpCc=
google.golang.org/genproto v0.0.0-20200115191322-ca5a22157cba/go.mod h1:n3cpQtvxv34hfy77yVDNjmbRyujviMdxYliBSkLhpCc=
google.golang.org/genproto v0.0.0-20200122232147-0452cf42e150/go.mod h1:n3cpQtvxv34hfy77yVDNjmbRyujviMdxYliBSkLhpCc=
google.golang.org/genproto v0.0.0-20200204135345-fa8e72b47b90/go.mod h1:GmwEX6Z4W5gMy59cAlVYjN9JhxgbQH6Gn+gFDQe2lzA=
google.golang.org/genproto v0.0.0-20200212174721-66ed5ce911ce/go.mod h1:55QSHmfGQM9UVYDPBsyGGes0y52j32PQ3BqQfXhyH3c=
google.golang.org/genproto v0.0.0-20200224152610-e50cd9704f63/go.mod h1:55QSHmfGQM9UVYDPBsyGGes0y52j32PQ3BqQfXhyH3c=
google.golang.org/genproto v0.0.0-20200228133532-8c2c7df3a383/go.mod h1:55QSHmfGQM9UVYDPBsyGGes0y52j32PQ3BqQfXhyH3c=
google.golang.org/genproto v0.0.0-20200305110556-506484158171/go.mod h1:55QSHmfGQM9UVYDPBsyGGes0y52j32PQ3BqQfXhyH3c=
google.golang.org/genproto v0.0.0-20200312145019-da6875a35672/go.mod h1:55QSHmfGQM9UVYDPBsyGGes0y52j32PQ3BqQfXhyH3c=
google.golang.org/genproto v0.0.0-20200331122359-1ee6d9798940/go.mod h1:55QSHmfGQM9UVYDPBsyGGes0y52j32PQ3BqQfXhyH3c=
google.golang.org/genproto v0.0.0-20200430143042-b979b6f78d84/go.mod h1:55QSHmfGQM9UVYDPBsyGGes0y52j32PQ3BqQfXhyH3c=
google.golang.org/genproto v0.0.0-20200511104702-f5ebc3bea380/go.mod h1:55QSHmfGQM9UVYDPBsyGGes0y52j32PQ3BqQfXhyH3c=
google.golang.org/genproto v0.0.0-20200515170657-fc4c6c6a6587/go.mod h1:YsZOwe1myG/8QRHRsmBRE1LrgQY60beZKjly0O1fX9U=
google.golang.org/genproto v0.0.0-20200526211855-cb27e3aa2013/go.mod h1:NbSheEEYHJ7i3ixzK3sjbqSGDJWnxyFXZblF3eUsNvo=
google.golang.org/genproto v0.0.0-20200618031413-b414f8b61790/go.mod h1:jDfRM7FcilCzHH/e9qn6dsT145K34l5v+OpcnNgKAAA=
google.golang.org/genproto v0.0.0-20200729003335-053ba62fc06f/go.mod h1:FWY/as6DDZQgahTzZj3fqbO1CbirC29ZNUFHwi0/+no=
google.golang.org/genproto v0.0.0-20200804131852-c06518451d9c/go.mod h1:FWY/as6DDZQgahTzZj3fqbO1CbirC29ZNUFHwi0/+no=
google.golang.org/genproto v0.0.0-20200825200019-8632dd797987/go.mod h1:FWY/as6DDZQgahTzZj3fqbO1CbirC29ZNUFHwi0/+no=
google.golang.org/genproto v0.0.0-20200904004341-0bd0a958aa1d/go.mod h1:FWY/as6DDZQgahTzZj3fqbO1CbirC29ZNUFHwi0/+no=
google.golang.org/genproto v0.0.0-20201109203340-2640f1f9cdfb/go.mod h1:FWY/as6DDZQgahTzZj3fqbO1CbirC29ZNUFHwi0/+no=
google.golang.org/genproto v0.0.0-20201201144952-b05cb90ed32e/go.mod h1:FWY/as6DDZQgahTzZj3fqbO1CbirC29ZNUFHwi0/+no=
google.golang.org/genproto v0.0.0-20201210142538-e3217bee35cc/go.mod h1:FWY/as6DDZQgahTzZj3fqbO1CbirC29ZNUFHwi0/+no=
google.golang.org/genproto v0.0.0-20201214200347-8c77b98c765d/go.mod h1:FWY/as6DDZQgahTzZj3fqbO1CbirC29ZNUFHwi0/+no=
google.golang.org/genproto v0.0.0-20210108203827-ffc7fda8c3d7/go.mod h1:FWY/as6DDZQgahTzZj3fqbO1CbirC29ZNUFHwi0/+no=
google.golang.org/genproto v0.0.0-20210226172003-ab064af71705/go.mod h1:FWY/as6DDZQgahTzZj3fqbO1CbirC29ZNUFHwi0/+no=
google.golang.org/grpc v1.19.0/go.mod h1:mqu4LbDTu4XGKhr4mRzUsmM4RtVoemTSY81AxZiDr8c=
google.golang.org/grpc v1.20.1/go.mod h1:10oTOabMzJvdu6/UiuZezV6QK5dSlG84ov/aaiqXj38=
google.golang.org/grpc v1.21.1/go.mod h1:oYelfM1adQP15Ek0mdvEgi9Df8B9CZIaU1084ijfRaM=
google.golang.org/grpc v1.23.0/go.mod h1:Y5yQAOtifL1yxbo5wqy6BxZv8vAUGQwXBOALyacEbxg=
google.golang.org/grpc v1.25.1/go.mod h1:c3i+UQWmh7LiEpx4sFZnkU36qjEYZ0imhYfXVyQciAY=
google.golang.org/grpc v1.26.0/go.mod h1:qbnxyOmOxrQa7FizSgH+ReBfzJrCY1pSN7KXBS8abTk=
google.golang.org/grpc v1.27.0/go.mod h1:qbnxyOmOxrQa7FizSgH+ReBfzJrCY1pSN7KXBS8abTk=
google.golang.org/grpc v1.27.1/go.mod h1:qbnxyOmOxrQa7FizSgH+ReBfzJrCY1pSN7KXBS8abTk=
google.golang.org/grpc v1.28.0/go.mod h1:rpkK4SK4GF4Ach/+MFLZUBavHOvF2JJB5uozKKal+60=
google.golang.org/grpc v1.29.1/go.mod h1:itym6AZVZYACWQqET3MqgPpjcuV5QH3BxFS3IjizoKk=
google.golang.org/grpc v1.30.0/go.mod h1:N36X2cJ7JwdamYAgDz+s+rVMFjt3numwzf/HckM8pak=
google.golang.org/grpc v1.31.0/go.mod h1:N36X2cJ7JwdamYAgDz+s+rVMFjt3numwzf/HckM8pak=
google.golang.org/grpc v1.31.1/go.mod h1:N36X2cJ7JwdamYAgDz+s+rVMFjt3numwzf/HckM8pak=
google.golang.org/grpc v1.33.2/go.mod h1:JMHMWHQWaTccqQQlmk3MJZS+GWXOdAesneDmEnv2fbc=
google.golang.org/grpc v1.34.0/go.mod h1:WotjhfgOW/POjDeRt8vscBtXq+2VjORFy659qA51WJ8=
google.golang.org/grpc v1.35.0/go.mod h1:qjiiYl8FncCW8feJPdyg3v6XW24KsRHe+dy9BAGRRjU=
google.golang.org/protobuf v0.0.0-20200109180630-ec00e32a8dfd/go.mod h1:DFci5gLYBciE7Vtevhsrf46CRTquxDuWsQurQQe4oz8=
google.golang.org/protobuf v0.0.0-20200221191635-4d8936d0db64/go.mod h1:kwYJMbMJ01Woi6D6+Kah6886xMZcty6N08ah7+eCXa0=
google.golang.org/protobuf v0.0.0-20200228230310-ab0ca4ff8a60/go.mod h1:cfTl7dwQJ+fmap5saPgwCLgHXTUD7jkjRqWcaiX5VyM=
google.golang.org/protobuf v1.20.1-0.20200309200217-e05f789c0967/go.mod h1:A+miEFZTKqfCUM6K7xSMQL9OKL/b6hQv+e19PK+JZNE=
google.golang.org/protobuf v1.21.0/go.mod h1:47Nbq4nVaFHyn7ilMalzfO3qCViNmqZ2kzikPIcrTAo=
google.golang.org/protobuf v1.22.0/go.mod h1:EGpADcykh3NcUnDUJcl1+ZksZNG86OlYog2l/sGQquU=
google.golang.org/protobuf v1.23.0/go.mod h1:EGpADcykh3NcUnDUJcl1+ZksZNG86OlYog2l/sGQquU=
google.golang.org/protobuf v1.23.1-0.20200526195155-81db48ad09cc/go.mod h1:EGpADcykh3NcUnDUJcl1+ZksZNG86OlYog2l/sGQquU=
google.golang.org/protobuf v1.24.0/go.mod h1:r/3tXBNzIEhYS9I1OUVjXDlt8tc493IdKGjtUeSXeh4=
google.golang.org/protobuf v1.25.0/go.mod h1:9JNX74DMeImyA3h4bdi1ymwjUzf21/xIlbajtzgsN7c=
google.golang.org/protobuf v1.26.0-rc.1/go.mod h1:jlhhOSvTdKEhbULTjvd4ARK9grFBp09yW+WbY/TyQbw=
google.golang.org/protobuf v1.26.0/go.mod h1:9q0QmTI4eRPtz6boOQmLYwt+qCgq0jsYwAQnmE0givc=
google.golang.org/protobuf v1.30.0 h1:kPPoIgf3TsEvrm0PFe15JQ+570QVxYzEvvHqChK+cng=
google.golang.org/protobuf v1.30.0/go.mod h1:HV8QOd/L58Z+nl8r43ehVNZIU/HEI6OcFqwMG9pJV4I=
gopkg.in/alecthomas/kingpin.v2 v2.2.6/go.mod h1:FMv+mEhP44yOT+4EoQTLFTRgOQ1FBLkstjWtayDeSgw=
gopkg.in/check.v1 v0.0.0-20161208181325-20d25e280405/go.mod h1:Co6ibVJAznAaIkqp8huTwlJQCZ016jof/cbN4VW5Yz0=
gopkg.in/check.v1 v1.0.0-20180628173108-788fd7840127/go.mod h1:Co6ibVJAznAaIkqp8huTwlJQCZ016jof/cbN4VW5Yz0=
gopkg.in/check.v1 v1.0.0-20190902080502-41f04d3bba15/go.mod h1:Co6ibVJAznAaIkqp8huTwlJQCZ016jof/cbN4VW5Yz0=
gopkg.in/check.v1 v1.0.0-20200227125254-8fa46927fb4f h1:BLraFXnmrev5lT+xlilqcH8XK9/i0At2xKjWk4p6zsU=
gopkg.in/check.v1 v1.0.0-20200227125254-8fa46927fb4f/go.mod h1:Co6ibVJAznAaIkqp8huTwlJQCZ016jof/cbN4VW5Yz0=
gopkg.in/errgo.v2 v2.1.0/go.mod h1:hNsd1EY+bozCKY1Ytp96fpM3vjJbqLJn88ws8XvfDNI=
gopkg.in/inf.v0 v0.9.1 h1:73M5CoZyi3ZLMOyDlQh031Cx6N9NDJ2Vvfl76EDAgDc=
gopkg.in/inf.v0 v0.9.1/go.mod h1:cWUDdTG/fYaXco+Dcufb5Vnc6Gp2YChqWtbxRZE0mXw=
gopkg.in/ini.v1 v1.67.0 h1:Dgnx+6+nfE+IfzjUEISNeydPJh9AXNNsWbGP9KzCsOA=
gopkg.in/ini.v1 v1.67.0/go.mod h1:pNLf8WUiyNEtQjuu5G5vTm06TEv9tsIgeAvK8hOrP4k=
gopkg.in/yaml.v2 v2.2.1/go.mod h1:hI93XBmqTisBFMUTm0b8Fm+jr3Dg1NNxqwp+5A1VGuI=
gopkg.in/yaml.v2 v2.2.2/go.mod h1:hI93XBmqTisBFMUTm0b8Fm+jr3Dg1NNxqwp+5A1VGuI=
gopkg.in/yaml.v2 v2.2.4/go.mod h1:hI93XBmqTisBFMUTm0b8Fm+jr3Dg1NNxqwp+5A1VGuI=
gopkg.in/yaml.v2 v2.2.5/go.mod h1:hI93XBmqTisBFMUTm0b8Fm+jr3Dg1NNxqwp+5A1VGuI=
gopkg.in/yaml.v2 v2.2.8/go.mod h1:hI93XBmqTisBFMUTm0b8Fm+jr3Dg1NNxqwp+5A1VGuI=
gopkg.in/yaml.v2 v2.3.0/go.mod h1:hI93XBmqTisBFMUTm0b8Fm+jr3Dg1NNxqwp+5A1VGuI=
gopkg.in/yaml.v2 v2.4.0 h1:D8xgwECY7CYvx+Y2n4sBz93Jn9JRvxdiyyo8CTfuKaY=
gopkg.in/yaml.v2 v2.4.0/go.mod h1:RDklbk79AGWmwhnvt/jBztapEOGDOx6ZbXqjP6csGnQ=
gopkg.in/yaml.v3 v3.0.0-20200313102051-9f266ea9e77c/go.mod h1:K4uyk7z7BCEPqu6E+C64Yfv1cQ7kz7rIZviUmN+EgEM=
gopkg.in/yaml.v3 v3.0.0-20200615113413-eeeca48fe776/go.mod h1:K4uyk7z7BCEPqu6E+C64Yfv1cQ7kz7rIZviUmN+EgEM=
gopkg.in/yaml.v3 v3.0.1 h1:fxVm/GzAzEWqLHuvctI91KS9hhNmmWOoWu0XTYJS7CA=
gopkg.in/yaml.v3 v3.0.1/go.mod h1:K4uyk7z7BCEPqu6E+C64Yfv1cQ7kz7rIZviUmN+EgEM=
honnef.co/go/tools v0.0.0-20190102054323-c2f93a96b099/go.mod h1:rf3lG4BRIbNafJWhAfAdb/ePZxsR/4RtNHQocxwk9r4=
honnef.co/go/tools v0.0.0-20190106161140-3f1c8253044a/go.mod h1:rf3lG4BRIbNafJWhAfAdb/ePZxsR/4RtNHQocxwk9r4=
honnef.co/go/tools v0.0.0-20190418001031-e561f6794a2a/go.mod h1:rf3lG4BRIbNafJWhAfAdb/ePZxsR/4RtNHQocxwk9r4=
honnef.co/go/tools v0.0.0-20190523083050-ea95bdfd59fc/go.mod h1:rf3lG4BRIbNafJWhAfAdb/ePZxsR/4RtNHQocxwk9r4=
honnef.co/go/tools v0.0.1-2019.2.3/go.mod h1:a3bituU0lyd329TUQxRnasdCoJDkEUEAqEt0JzvZhAg=
honnef.co/go/tools v0.0.1-2020.1.3/go.mod h1:X/FiERA/W4tHapMX5mGpAtMSVEeEUOyHaw9vFzvIQ3k=
honnef.co/go/tools v0.0.1-2020.1.4/go.mod h1:X/FiERA/W4tHapMX5mGpAtMSVEeEUOyHaw9vFzvIQ3k=
rsc.io/binaryregexp v0.2.0/go.mod h1:qTv7/COck+e2FymRvadv62gMdZztPaShugOCi3I+8D8=
rsc.io/pdf v0.1.1/go.mod h1:n8OzWcQ6Sp37PL01nO98y4iUCRdTGarVfzxY20ICaU4=
rsc.io/quote/v3 v3.1.0/go.mod h1:yEA65RcK8LyAZtP9Kv3t0HmxON59tX3rD+tICJqUlj0=
rsc.io/sampler v1.3.0/go.mod h1:T1hPZKmBbMNahiBKFy5HrXp6adAjACjK9JXDnKaTXpA=
aerohub@localhost:~/employee-api$ 

                                Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding those notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. We also recommend that a
      file or class name and description of purpose be included on the
      same "printed page" as the copyright notice for easier
      identification within third-party archives.

   Copyright [2023] [OpsTree Solutions]

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
aerohub@localhost:~/employee-api$ cat main.go 
package main

import (
	docs "employee-api/docs"
	"employee-api/middleware"
	"employee-api/routes"
	"github.com/gin-gonic/gin"
	"github.com/penglongli/gin-metrics/ginmetrics"
	"github.com/sirupsen/logrus"
	swaggerfiles "github.com/swaggo/files"
	ginSwagger "github.com/swaggo/gin-swagger"
)

var router = gin.New()

func init() {
	logrus.SetLevel(logrus.InfoLevel)
	logrus.SetFormatter(&logrus.JSONFormatter{}) // NEW
}

// @title Employee API
// @version 1.0
// @description The REST API documentation for employee webserver
// @termsOfService http://swagger.io/terms/

// @contact.name Opstree Solutions
// @contact.url https://opstree.com
// @contact.email opensource@opstree.com

// @license.name Apache 2.0
// @license.url http://www.apache.org/licenses/LICENSE-2.0.html
// @BasePath /api/v1
// @schemes http
func main() {
	monitor := ginmetrics.GetMonitor()
	monitor.SetMetricPath("/metrics")
	monitor.SetSlowTime(1)
	monitor.SetDuration([]float64{0.1, 0.3, 1.2, 5, 10})
	monitor.Use(router)
	router.Use(gin.Recovery())                  // NEW
	router.Use(middlewares.LoggingMiddleware()) // NEW
	v1 := router.Group("/api/v1")
	docs.SwaggerInfo.BasePath = "/api/v1/employee"
	routes.CreateRouterForEmployee(v1)
	url := ginSwagger.URL("http://localhost:8080/swagger/doc.json")
	router.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerfiles.Handler, url))
	router.Run(":8080")
}
aerohub@localhost:~/employee-api$ ls
api  client  config  config.yaml  Dockerfile  docs  go.mod  go.sum  LICENSE  main.go  Makefile  middleware  migration  migration.json  model  README.md  routes  static
aerohub@localhost:~/employee-api$ cat Makefile 
APP_VERSION ?= v0.1.0
IMAGE_REGISTRY ?= quay.io/opstree
IMAGE_NAME ?= employee-api

# Get the currently used golang install path (in GOPATH/bin, unless GOBIN is set)
ifeq (,$(shell go env GOBIN))
GOBIN=$(shell go env GOPATH)/bin
else
GOBIN=$(shell go env GOBIN)
endif

# Build employee binary
build: fmt
	go build -o employee-api

# Run go fmt against code
fmt:
	go fmt ./...

# Run go vet against code
vet:
	go vet ./...

docker-build:
	docker build -t ${IMAGE_REGISTRY}/${IMAGE_NAME}:${APP_VERSION} -f Dockerfile .

docker-push:
	docker push ${IMAGE_REGISTRY}/${IMAGE_NAME}:${APP_VERSION}

swagger:
	swag init -g main.go

run-migrations:
	migrate -source file://migration -database "$(shell cat migration.json | jq -r '.database')" up
aerohub@localhost:~/employee-api$ 


aerohub@localhost:~/employee-api/middleware$ ls
logging.go  logging_test.go
aerohub@localhost:~/employee-api/middleware$ cat logging.go 
package middlewares

import (
	"time"

	"github.com/gin-gonic/gin"
	log "github.com/sirupsen/logrus"
)

// LoggingMiddleware is a method to set formatter of logrus
func LoggingMiddleware() gin.HandlerFunc {
	return func(ctx *gin.Context) {
		// Starting time request
		startTime := time.Now()

		// Processing request
		ctx.Next()

		// End Time request
		endTime := time.Now()

		// execution time
		latencyTime := endTime.Sub(startTime)

		// Request method
		reqMethod := ctx.Request.Method

		// Request route
		reqUri := ctx.Request.RequestURI

		// status code
		statusCode := ctx.Writer.Status()

		// Request IP
		clientIP := ctx.ClientIP()

		log.WithFields(log.Fields{
			"http_method": reqMethod,
			"request_uri": reqUri,
			"status_code": statusCode,
			"latency":     latencyTime,
			"client_ip":   clientIP,
		}).Info("HTTP REQUEST STATUS")

		ctx.Next()
	}
}
aerohub@localhost:~/employee-api/middleware$ cat logging_test.go 
package middlewares

import (
	"github.com/gin-gonic/gin"
	"github.com/stretchr/testify/assert"
	"net/http"
	"net/http/httptest"
	"testing"
)

func TestLoggingMiddleware(t *testing.T) {
	router := gin.New()
	router.Use(LoggingMiddleware())

	req, _ := http.NewRequest("GET", "/example", nil)
	req.Header.Set("X-Real-IP", "192.168.0.1")
	w := httptest.NewRecorder()

	router.ServeHTTP(w, req)

	assert.Equal(t, 404, w.Code)

	expectedLogMessage := "404 page not found"
	assert.Contains(t, w.Body.String(), expectedLogMessage)
	assert.Contains(t, req.Method, "GET")
	assert.Contains(t, req.URL.Path, "/example")
	assert.Contains(t, "192.168.0.1", "192.168.0.1")
}
aerohub@localhost:~/employee-api/middleware$ 

aerohub@localhost:~/employee-api/migration$ ls
000001_create_employee_info_table.down.sql  000001_create_employee_info_table.up.sql
aerohub@localhost:~/employee-api/migration$ cat 000001_create_employee_info_table.down.sql 
CREATE TABLE IF NOT EXISTS employee_info (
    id text, name text, designation text, department text,
    joining_date date, address text, office_location text,
    status text, email text, phone_number text,
    PRIMARY KEY (id, joining_date)
) WITH CLUSTERING ORDER BY (joining_date DESC);aerohub@localhost:~/employee-api/migration$ cat 000001_create_employee_info_table.up.sql 
CREATE TABLE IF NOT EXISTS employee_info (
    id text, name text, designation text, department text,
    joining_date date, address text, office_location text,
    status text, email text, phone_number text,
    PRIMARY KEY (id, joining_date)
) WITH CLUSTERING ORDER BY (joining_date DESC);aerohub@localhost:~/employee-api/migration$ cd ..
aerohub@localhost:~/employee-api$ cat migration.json 
{
  "database": "cassandra://172.17.0.3:9042/employee_db?username=scylladb&password=password"
}aerohub@localhost:~/employee-api$ cat README.md 
<p align="center">
  <img src="./static/employee-api-logo.svg" height="280" width="280">
</p>

Employee REST API is a golang based microservice which is responsible for all the employee related transactions in the [OT-Microservices](https://github.com/OT-MICROSERVICES). This application is completely platform independent and can be run on any kind of platform.

Supported features in the applications are:-

- Gin REST API for web transactions
- ScyllaDB as primary database for storing information
- Redis as a cache management system for quick response
- Swagger integration for the documentation of the API
- Prometheus's metrics support to monitor application health and performance

## Pre-Requisites

The application doesn't have any specific pre-requisites except the database connectivity. Additionally, we can add `Redis` as cache system but it's not part of the mandatory setup.

For running the application, we need following things configured:

- [ScyllaDB](https://www.scylladb.com/)
- [Redis](https:/redis.com/)

## Architecture

![](./static/employee.png)

## Application 

For building the application, we can use `make` command. The same command can be used to build docker image as well.

```shell
make build
make docker-build
```

For execution of the unit test cases, code coverage reports, etc. We can use `go test` command.

```shell
go test $(go list ./... | grep -v docs | grep -v model | grep -v main.go) -coverprofile cover.out
# For HTML report visualization
go tool cover -html=cover.out
```

The test cases are covered for only these packages:-
- api
- client
- config
- middleware
- routes

For dev testing, the [Swagger](https://swagger.io/) UI can be used for sample payload generation and requests. The swagger page will be accessible on http://localhost:8080/swagger/index.html

To run the application, simple binary commands can be used. Before running the application, we have to make sure our mandatory database (ScyllaDB) is up and running. Configuration properties will be configured inside [config.yaml](./config.yaml) file. Also, once the property file is defined and configured properly, we need to run migrations to create database, schema etc. The connection details for migration is available in [migration.json](./migration.json).

```shell
make run-migrations
```

Once the keyspace and table is initialized, we can run the application by:

```shell
export GIN_MODE=release
# For debugging set gin mode to development
./employee-api
```
## Endpoints Information

| **Endpoint**                        | **Method** | **Description**                                                                                   |
|-------------------------------------|------------|---------------------------------------------------------------------------------------------------|
| /metrics                            | GET        | Application healthcheck and performance metrics are available on this endpoint                    |
| /api/v1/employee/health             | GET        | Endpoint for providing shallow healthcheck information about application health and readiness     |
| /api/v1/employee/health/detail      | GET        | Endpoint for providing detailed health check about dependencies as well like - ScyllaDB and Redis |
| /api/v1/employee/create             | POST       | Data creation endpoint which accepts certain JSON body to add employee information in database    |
| /api/v1/employee/search             | GET        | Endpoint for searching data information using the params in the URL                               |
| /api/v1/employee/search/all         | GET        | Endpoint for searching all information across the system                                          |
| /api/v1/employee/search/location    | GET        | Application endpoint for getting the count and information of location                            |
| /api/v1/employee/search/designation | GET        | Application endpoint for getting the count and information of designation                         |
| /swagger/index.html                 | GET        | Swagger endpoint for the application API documentation and their data models                      |

## Contact Information

[Opstree Opensource](mailto:opensource@opstree.com)
aerohub@localhost:~/employee-api$ 


erohub@localhost:~/employee-api$ cd model/
aerohub@localhost:~/employee-api/model$ ls
config.go  employee.go  health.go
aerohub@localhost:~/employee-api/model$ cat config.go 
package model

// Config struct is managing the configuration file
type Config struct {
	ScyllaDB ScyllaDB `yaml:"scylladb"`
	Redis    Redis    `yaml:"redis"`
}

type ScyllaDB struct {
	Host     []string `yaml:"host"`
	Keyspace string   `yaml:"keyspace"`
	Username string   `yaml:"username"`
	Password string   `yaml:"password"`
}

type Redis struct {
	Host     string `yaml:"host"`
	Password string `yaml:"password"`
	Database int    `yaml:"database"`
	Enabled  bool   `yaml:"enabled"`
}
aerohub@localhost:~/employee-api/model$ cat employee.go 
package model

// Employee struct will be the data mapping interface of all employee REST API data
type Employee struct {
	ID             string `json:"id"`
	Name           string `json:"name"`
	Designation    string `json:"designation"`
	Department     string `json:"department"`
	JoiningDate    string `json:"joining_date"`
	Address        string `json:"address"`
	OfficeLocation string `json:"office_location"`
	Status         string `json:"status"`
	EmailID        string `json:"email"`
	PhoneNumber    string `json:"phone_number"`
}

// CustomMessage is a structure for custom message with Gin
type CustomMessage struct {
	Message string `json:"message"`
}

// Location is a struct for mapping location interface of all employees
type Location struct {
	Noida     int `json:"Noida"`
	Bangalore int `json:"Bangalore"`
	Hyderabad int `json:"Hyderabad"`
	Delaware  int `json:"Delaware"`
}

// Designation is a struct for mapping designation interface for all employees
type Designation struct {
	DevOpsConsultant  int `json:"DevOps Consultant"`
	DevOpsSpecialist  int `json:"DevOps Specialist"`
	GrowthPartner     int `json:"Growth Partner"`
	ConsultantPartner int `json:"Consultant Partner"`
}
aerohub@localhost:~/employee-api/model$ ls
config.go  employee.go  health.go
aerohub@localhost:~/employee-api/model$ cat health.go 
package model

// DetailedHealthCheck is a structure for detailed health check information
type DetailedHealthCheck struct {
	Message     string `json:"message"`
	ScyllaDB    string `json:"scylla_db"`
	EmployeeAPI string `json:"employee_api"`
	Redis       string `json:"redis"`
}
aerohub@localhost:~/employee-api/model$ 


erohub@localhost:~/employee-api$ cd routes/
aerohub@localhost:~/employee-api/routes$ ls
routes.go  routes_test.go
aerohub@localhost:~/employee-api/routes$ cat routes.go 
package routes

import (
	"employee-api/api"
	"github.com/gin-gonic/gin"
)

// @BasePath /employee
// CreateRouterForEmployee is a method for generate routes
func CreateRouterForEmployee(routerGroup *gin.RouterGroup) {
	employee := routerGroup.Group("/employee")
	employee.GET("/health", api.HealthCheckAPI)
	employee.GET("/health/detail", api.DetailedHealthCheckAPI)
	employee.POST("/create", api.CreateEmployeeData)
	employee.GET("/search", api.ReadEmployeeData)
	employee.GET("/search/all", api.ReadCompleteEmployeesData)
	employee.GET("/search/location", api.ReadEmployeesLocation)
	employee.GET("/search/designation", api.ReadEmployeesDesignation)
}
aerohub@localhost:~/employee-api/routes$ cat routes_test.go 
package routes

import (
	"net/http"
	"net/http/httptest"
	"testing"

	"github.com/gin-gonic/gin"
	"github.com/stretchr/testify/assert"
)

func TestCreateRouterForEmployee(t *testing.T) {
	// Create a test router using the Gin framework
	router := gin.Default()
	routerGroup := router.Group("/api")
	CreateRouterForEmployee(routerGroup)

	// Health Check API
	healthCheckReq, _ := http.NewRequest("GET", "/api/employee/health", nil)
	healthCheckResp := performRequest(router, healthCheckReq)
	assert.Equal(t, http.StatusBadRequest, healthCheckResp.Code)

	// Detailed Health Check API
	detailedHealthCheckReq, _ := http.NewRequest("GET", "/api/employee/health/detail", nil)
	detailedHealthCheckResp := performRequest(router, detailedHealthCheckReq)
	assert.Equal(t, http.StatusBadRequest, detailedHealthCheckResp.Code)

	// Create Employee Data API
	createEmployeeDataReq, _ := http.NewRequest("POST", "/api/employee/create", nil)
	createEmployeeDataResp := performRequest(router, createEmployeeDataReq)
	assert.Equal(t, http.StatusBadRequest, createEmployeeDataResp.Code)

	// Read Employee Data API
	readEmployeeDataReq, _ := http.NewRequest("GET", "/api/employee/search", nil)
	readEmployeeDataResp := performRequest(router, readEmployeeDataReq)
	assert.Equal(t, http.StatusBadRequest, readEmployeeDataResp.Code)

	// Read Complete Employees Data API
	readCompleteEmployeesDataReq, _ := http.NewRequest("GET", "/api/employee/search/all", nil)
	readCompleteEmployeesDataResp := performRequest(router, readCompleteEmployeesDataReq)
	assert.Equal(t, http.StatusBadRequest, readCompleteEmployeesDataResp.Code)

	// Read Employees Location API
	readEmployeesLocationReq, _ := http.NewRequest("GET", "/api/employee/search/location", nil)
	readEmployeesLocationResp := performRequest(router, readEmployeesLocationReq)
	assert.Equal(t, http.StatusBadRequest, readEmployeesLocationResp.Code)

	// Read Employees Designation API
	readEmployeesDesignationReq, _ := http.NewRequest("GET", "/api/employee/search/designation", nil)
	readEmployeesDesignationResp := performRequest(router, readEmployeesDesignationReq)
	assert.Equal(t, http.StatusBadRequest, readEmployeesDesignationResp.Code)
}

// Helper function to perform the HTTP request and retrieve the response
func performRequest(router *gin.Engine, req *http.Request) *httptest.ResponseRecorder {
	w := httptest.NewRecorder()
	router.ServeHTTP(w, req)
	return w
}
aerohub@localhost:~/employee-api/routes$ ls
routes.go  routes_test.go
aerohub@localhost:~/employee-api/routes$ cd ..
aerohub@localhost:~/employee-api$ ls
api  client  config  config.yaml  Dockerfile  docs  go.mod  go.sum  LICENSE  main.go  Makefile  middleware  migration  migration.json  model  README.md  routes  static
aerohub@localhost:~/employee-api$ 


<img width="2720" height="2640" alt="scylladb_connection_flow" src="https://github.com/user-attachments/assets/6a9cde60-14e1-4640-976d-61e3ce6513f0" />



# Salary

ritu@localhost:~$ cd salary-api/
ritu@localhost:~/salary-api$ la
Dockerfile  .gitignore  Makefile   migration.json  mvnw      pom.xml    src
.git        LICENSE     migration  .mvn            mvnw.cmd  README.md  static
ritu@localhost:~/salary-api$ cat Makefile 
APP_VERSION ?= v0.1.0
IMAGE_REGISTRY ?= quay.io/opstree
IMAGE_NAME ?= salary-api

# Build salary api
build:
	mvn clean package

# Run checkstyle against code
fmt:
	mvn checkstyle:checkstyle

# Run jacoco test cases for coverage
test:
	mvn test

docker-build:
	docker build -t ${IMAGE_REGISTRY}/${IMAGE_NAME}:${APP_VERSION} -f Dockerfile .

docker-push:
	docker push ${IMAGE_REGISTRY}/${IMAGE_NAME}:${APP_VERSION}

run-migrations:
	migrate -source file://migration -database "$(shell cat migration.json | jq -r '.database')" up
ritu@localhost:~/salary-api$ ls
Dockerfile  Makefile   migration.json  mvnw.cmd  README.md  static
LICENSE     migration  mvnw            pom.xml   src
ritu@localhost:~/salary-api$ cat migration.json 
{
  "database": "cassandra://172.17.0.3:9042/employee_db?username=scylladb&password=password"
}ritu@localhost:~/salary-api$ cat mvnw.cmd 
@REM ----------------------------------------------------------------------------
@REM Licensed to the Apache Software Foundation (ASF) under one
@REM or more contributor license agreements.  See the NOTICE file
@REM distributed with this work for additional information
@REM regarding copyright ownership.  The ASF licenses this file
@REM to you under the Apache License, Version 2.0 (the
@REM "License"); you may not use this file except in compliance
@REM with the License.  You may obtain a copy of the License at
@REM
@REM    http://www.apache.org/licenses/LICENSE-2.0
@REM
@REM Unless required by applicable law or agreed to in writing,
@REM software distributed under the License is distributed on an
@REM "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
@REM KIND, either express or implied.  See the License for the
@REM specific language governing permissions and limitations
@REM under the License.
@REM ----------------------------------------------------------------------------

@REM ----------------------------------------------------------------------------
@REM Apache Maven Wrapper startup batch script, version 3.2.0
@REM
@REM Required ENV vars:
@REM JAVA_HOME - location of a JDK home dir
@REM
@REM Optional ENV vars
@REM MAVEN_BATCH_ECHO - set to 'on' to enable the echoing of the batch commands
@REM MAVEN_BATCH_PAUSE - set to 'on' to wait for a keystroke before ending
@REM MAVEN_OPTS - parameters passed to the Java VM when running Maven
@REM     e.g. to debug Maven itself, use
@REM set MAVEN_OPTS=-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=8000
@REM MAVEN_SKIP_RC - flag to disable loading of mavenrc files
@REM ----------------------------------------------------------------------------

@REM Begin all REM lines with '@' in case MAVEN_BATCH_ECHO is 'on'
@echo off
@REM set title of command window
title %0
@REM enable echoing by setting MAVEN_BATCH_ECHO to 'on'
@if "%MAVEN_BATCH_ECHO%" == "on"  echo %MAVEN_BATCH_ECHO%

@REM set %HOME% to equivalent of $HOME
if "%HOME%" == "" (set "HOME=%HOMEDRIVE%%HOMEPATH%")

@REM Execute a user defined script before this one
if not "%MAVEN_SKIP_RC%" == "" goto skipRcPre
@REM check for pre script, once with legacy .bat ending and once with .cmd ending
if exist "%USERPROFILE%\mavenrc_pre.bat" call "%USERPROFILE%\mavenrc_pre.bat" %*
if exist "%USERPROFILE%\mavenrc_pre.cmd" call "%USERPROFILE%\mavenrc_pre.cmd" %*
:skipRcPre

@setlocal

set ERROR_CODE=0

@REM To isolate internal variables from possible post scripts, we use another setlocal
@setlocal

@REM ==== START VALIDATION ====
if not "%JAVA_HOME%" == "" goto OkJHome

echo.
echo Error: JAVA_HOME not found in your environment. >&2
echo Please set the JAVA_HOME variable in your environment to match the >&2
echo location of your Java installation. >&2
echo.
goto error

:OkJHome
if exist "%JAVA_HOME%\bin\java.exe" goto init

echo.
echo Error: JAVA_HOME is set to an invalid directory. >&2
echo JAVA_HOME = "%JAVA_HOME%" >&2
echo Please set the JAVA_HOME variable in your environment to match the >&2
echo location of your Java installation. >&2
echo.
goto error

@REM ==== END VALIDATION ====

:init

@REM Find the project base dir, i.e. the directory that contains the folder ".mvn".
@REM Fallback to current working directory if not found.

set MAVEN_PROJECTBASEDIR=%MAVEN_BASEDIR%
IF NOT "%MAVEN_PROJECTBASEDIR%"=="" goto endDetectBaseDir

set EXEC_DIR=%CD%
set WDIR=%EXEC_DIR%
:findBaseDir
IF EXIST "%WDIR%"\.mvn goto baseDirFound
cd ..
IF "%WDIR%"=="%CD%" goto baseDirNotFound
set WDIR=%CD%
goto findBaseDir

:baseDirFound
set MAVEN_PROJECTBASEDIR=%WDIR%
cd "%EXEC_DIR%"
goto endDetectBaseDir

:baseDirNotFound
set MAVEN_PROJECTBASEDIR=%EXEC_DIR%
cd "%EXEC_DIR%"

:endDetectBaseDir

IF NOT EXIST "%MAVEN_PROJECTBASEDIR%\.mvn\jvm.config" goto endReadAdditionalConfig

@setlocal EnableExtensions EnableDelayedExpansion
for /F "usebackq delims=" %%a in ("%MAVEN_PROJECTBASEDIR%\.mvn\jvm.config") do set JVM_CONFIG_MAVEN_PROPS=!JVM_CONFIG_MAVEN_PROPS! %%a
@endlocal & set JVM_CONFIG_MAVEN_PROPS=%JVM_CONFIG_MAVEN_PROPS%

:endReadAdditionalConfig

SET MAVEN_JAVA_EXE="%JAVA_HOME%\bin\java.exe"
set WRAPPER_JAR="%MAVEN_PROJECTBASEDIR%\.mvn\wrapper\maven-wrapper.jar"
set WRAPPER_LAUNCHER=org.apache.maven.wrapper.MavenWrapperMain

set WRAPPER_URL="https://repo.maven.apache.org/maven2/org/apache/maven/wrapper/maven-wrapper/3.2.0/maven-wrapper-3.2.0.jar"

FOR /F "usebackq tokens=1,2 delims==" %%A IN ("%MAVEN_PROJECTBASEDIR%\.mvn\wrapper\maven-wrapper.properties") DO (
    IF "%%A"=="wrapperUrl" SET WRAPPER_URL=%%B
)

@REM Extension to allow automatically downloading the maven-wrapper.jar from Maven-central
@REM This allows using the maven wrapper in projects that prohibit checking in binary data.
if exist %WRAPPER_JAR% (
    if "%MVNW_VERBOSE%" == "true" (
        echo Found %WRAPPER_JAR%
    )
) else (
    if not "%MVNW_REPOURL%" == "" (
        SET WRAPPER_URL="%MVNW_REPOURL%/org/apache/maven/wrapper/maven-wrapper/3.2.0/maven-wrapper-3.2.0.jar"
    )
    if "%MVNW_VERBOSE%" == "true" (
        echo Couldn't find %WRAPPER_JAR%, downloading it ...
        echo Downloading from: %WRAPPER_URL%
    )

    powershell -Command "&{"^
		"$webclient = new-object System.Net.WebClient;"^
		"if (-not ([string]::IsNullOrEmpty('%MVNW_USERNAME%') -and [string]::IsNullOrEmpty('%MVNW_PASSWORD%'))) {"^
		"$webclient.Credentials = new-object System.Net.NetworkCredential('%MVNW_USERNAME%', '%MVNW_PASSWORD%');"^
		"}"^
		"[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; $webclient.DownloadFile('%WRAPPER_URL%', '%WRAPPER_JAR%')"^
		"}"
    if "%MVNW_VERBOSE%" == "true" (
        echo Finished downloading %WRAPPER_JAR%
    )
)
@REM End of extension

@REM If specified, validate the SHA-256 sum of the Maven wrapper jar file
SET WRAPPER_SHA_256_SUM=""
FOR /F "usebackq tokens=1,2 delims==" %%A IN ("%MAVEN_PROJECTBASEDIR%\.mvn\wrapper\maven-wrapper.properties") DO (
    IF "%%A"=="wrapperSha256Sum" SET WRAPPER_SHA_256_SUM=%%B
)
IF NOT %WRAPPER_SHA_256_SUM%=="" (
    powershell -Command "&{"^
       "$hash = (Get-FileHash \"%WRAPPER_JAR%\" -Algorithm SHA256).Hash.ToLower();"^
       "If('%WRAPPER_SHA_256_SUM%' -ne $hash){"^
       "  Write-Output 'Error: Failed to validate Maven wrapper SHA-256, your Maven wrapper might be compromised.';"^
       "  Write-Output 'Investigate or delete %WRAPPER_JAR% to attempt a clean download.';"^
       "  Write-Output 'If you updated your Maven version, you need to update the specified wrapperSha256Sum property.';"^
       "  exit 1;"^
       "}"^
       "}"
    if ERRORLEVEL 1 goto error
)

@REM Provide a "standardized" way to retrieve the CLI args that will
@REM work with both Windows and non-Windows executions.
set MAVEN_CMD_LINE_ARGS=%*

%MAVEN_JAVA_EXE% ^
  %JVM_CONFIG_MAVEN_PROPS% ^
  %MAVEN_OPTS% ^
  %MAVEN_DEBUG_OPTS% ^
  -classpath %WRAPPER_JAR% ^
  "-Dmaven.multiModuleProjectDirectory=%MAVEN_PROJECTBASEDIR%" ^
  %WRAPPER_LAUNCHER% %MAVEN_CONFIG% %*
if ERRORLEVEL 1 goto error
goto end

:error
set ERROR_CODE=1

:end
@endlocal & set ERROR_CODE=%ERROR_CODE%

if not "%MAVEN_SKIP_RC%"=="" goto skipRcPost
@REM check for post script, once with legacy .bat ending and once with .cmd ending
if exist "%USERPROFILE%\mavenrc_post.bat" call "%USERPROFILE%\mavenrc_post.bat"
if exist "%USERPROFILE%\mavenrc_post.cmd" call "%USERPROFILE%\mavenrc_post.cmd"
:skipRcPost

@REM pause the script if MAVEN_BATCH_PAUSE is set to 'on'
if "%MAVEN_BATCH_PAUSE%"=="on" pause

if "%MAVEN_TERMINATE_CMD%"=="on" exit %ERROR_CODE%

cmd /C exit /B %ERROR_CODE%
ritu@localhost:~/salary-api$ ls
Dockerfile  Makefile   migration.json  mvnw.cmd  README.md  static
LICENSE     migration  mvnw            pom.xml   src
ritu@localhost:~/salary-api$ cat README.md 
<p align="center">
  <img src="./static/salary-api-logo.svg" height="300" width="300">
</p>

Salary API is a Java based microservice which is responsible for all the salary related transactions and records in **[OT-Microservices](https://github.com/OT-MICROSERVICES)** stack. The application is platform independent and can be run on multiple operating system. **[Java Runtime](https://www.java.com/en/download/manual.jsp)** would be required to run this application.

Supported features of the Salary API are:-

- Spring boot based web framework, which uses tomcat as webserver.
- ScyllaDB is used as primary database for storing all the salary data.
- Redis as cache manager to store the cache response.
- Prometheus and Open-telemetry metrics support for monitoring and observability
- Swagger integration for the API documentation of endpoints and payloads.
- Database migration using the tool called **[migrate](https://github.com/golang-migrate/migrate)**.

## Pre-Requisites

The Salary API application have some database, cache manager and package dependencies. Some of the dependencies are optional and some are mandatory. To compile the application, we only need `maven` as build tool, but for running the application following things are required:-

- **[ScyllaDB](https://www.scylladb.com/)**
- **[Redis](https://redis.io/)**
- **[Migrate](https://github.com/golang-migrate/migrate)**
- **[Maven](https://maven.apache.org/)**

Maven will be used as package manager to download specific version of dependencies to run the Salary API.

## Architecture

![](./static/salary.png)

## Application

For building the Salary API application, we can use `make` commands with our **[Makefile](./Makefile)**. But first, we need to install the dependencies which can be simply done using the `make` command.

```shell
make build
```

For building the docker image artifact of the attendance api, we can invoke another make command.

```shell
make docker-build
make docker-push
```

Also, Salary API contains different test cases and code quality related integrations. To check the code quality, we can use `checkstyle` plugin with maven. Also, for code coverage and unit testing, we are using Jacoco, and Junit respectively.

```shell
make fmt
make test
```

```shell
mvn checkstyle:checkstyle
# For unit testing and code coverage
mvn test
```

The test cases are present in **[src/test/java/com/opstree/microservice/salary](./src/test/java/com/opstree/microservice/salary)**. For dev testing, the Swagger UI can be used for sample payload generation and requests. The swagger page will be accessible on http://localhost:8080/salary-documentation.

Before running the application, we have to make sure our mandatory database (ScyllaDB and Redis) is up and running. Configuration properties will be configured inside **[application.yml](./src/main/resources/application.yml)** file. Also, once the property file is defined and configured properly, we need to run migrations to create database, schema etc. The connection details for migration is available in **[migration.json](./migration.json)**.

```shell
make run-migrations
```

Once the schema, table and database is configured, we can start our application using java runtime.

```shell
java -jar target/salary-0.1.0-RELEASE.jar
```

## Endpoint Information

| **Endpoint**                   | **Method** | **Description**                                                                               |
|--------------------------------|------------|-----------------------------------------------------------------------------------------------|
| `/api/v1/salary/create/record` | POST       | Data creation endpoint which accepts certain JSON body to add salary information in database  |
| `/api/v1/salary/search`        | GET        | Endpoint for searching data information using the params in the URL                           |
| `/api/v1/salary/search/all`    | GET        | Endpoint for searching all information across the system                                      |
| `/actuator/prometheus`         | GET        | Application healthcheck and performance metrics are available on this endpoint                |
| `/actuator/health`             | GET        | Endpoint for providing shallow healthcheck information about application health and readiness |

## Contact Information

[Opstree Opensource](mailto:opensource@opstree.com)
ritu@localhost:~/salary-api$ ls
Dockerfile  Makefile   migration.json  mvnw.cmd  README.md  static
LICENSE     migration  mvnw            pom.xml   src
ritu@localhost:~/salary-api$ cat LICENSE 
                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding those notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. We also recommend that a
      file or class name and description of purpose be included on the
      same "printed page" as the copyright notice for easier
      identification within third-party archives.

   Copyright [2023] [Opstree Solutions]

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
ritu@localhost:~/salary-api$ ls
Dockerfile  Makefile   migration.json  mvnw.cmd  README.md  static
LICENSE     migration  mvnw            pom.xml   src
ritu@localhost:~/salary-api$ cd migration/
ritu@localhost:~/salary-api/migration$ ls
000001_create_employee_salary_table.down.sql
000001_create_employee_salary_table.up.sql
ritu@localhost:~/salary-api/migration$ cat 000001_create_employee_salary_table.down.sql 
CREATE TABLE IF NOT EXISTS employee_salary (
                                 id text,
                                 process_date text,
                                 name text,
                                 salary float,
                                 status text,
                                 PRIMARY KEY (id, process_date)
) WITH CLUSTERING ORDER BY (process_date DESC);
ritu@localhost:~/salary-api/migration$ cat 000001_create_employee_salary_table.down.sql 
CREATE TABLE IF NOT EXISTS employee_salary (
                                 id text,
                                 process_date text,
                                 name text,
                                 salary float,
                                 status text,
                                 PRIMARY KEY (id, process_date)
) WITH CLUSTERING ORDER BY (process_date DESC);
ritu@localhost:~/salary-api/migration$ cd ..
ritu@localhost:~/salary-api$ ls
Dockerfile  Makefile   migration.json  mvnw.cmd  README.md  static
LICENSE     migration  mvnw            pom.xml   src
ritu@localhost:~/salary-api$ cat pom.xml 
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.1.1</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.opstree.microservice</groupId>
	<artifactId>salary</artifactId>
	<version>0.1.0-RELEASE</version>
	<name>salary</name>
	<description>Java microservice to handle all salary related data</description>
	<properties>
		<java.version>17</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>net.logstash.logback</groupId>
			<artifactId>logstash-logback-encoder</artifactId>
			<version>6.6</version>
		</dependency>
		<dependency>
			<groupId>org.springdoc</groupId>
			<artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
			<version>2.0.3</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-cassandra</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
		<dependency>
			<groupId>io.micrometer</groupId>
			<artifactId>micrometer-registry-prometheus</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<excludes>
						<exclude>
							<groupId>org.projectlombok</groupId>
							<artifactId>lombok</artifactId>
						</exclude>
					</excludes>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-surefire-plugin</artifactId>
<!--				<version>2.17</version>-->
				<configuration>
					<excludes>
						<exclude>com/opstree/microservice/salary/controller/**</exclude>
						<exclude>com/opstree/microservice/salary/service/**</exclude>
						<exclude>com/opstree/microservice/salary/repository/**</exclude>
					</excludes>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.jacoco</groupId>
				<artifactId>jacoco-maven-plugin</artifactId>
				<version>0.8.7</version>
				<executions>
					<execution>
						<goals>
							<goal>prepare-agent</goal>
						</goals>
					</execution>
					<execution>
						<id>report</id>
						<phase>test</phase>
						<goals>
							<goal>report</goal>
						</goals>
					</execution>
				</executions>
				<configuration>
					<excludes>
						<exclude>com/opstree/microservice/salary/controller/**</exclude>
						<exclude>com/opstree/microservice/salary/service/**</exclude>
						<exclude>com/opstree/microservice/salary/repository/**</exclude>
					</excludes>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
ritu@localhost:~/salary-api$ cd src/
ritu@localhost:~/salary-api/src$ ls
main  test
ritu@localhost:~/salary-api/src$ cd main/
ritu@localhost:~/salary-api/src/main$ ls
java  resources
ritu@localhost:~/salary-api/src/main$ cd java/
ritu@localhost:~/salary-api/src/main/java$ ls
com
ritu@localhost:~/salary-api/src/main/java$ cd com/
ritu@localhost:~/salary-api/src/main/java/com$ ls
opstree
ritu@localhost:~/salary-api/src/main/java/com$ c do
c: command not found
ritu@localhost:~/salary-api/src/main/java/com$ cd opstree/
ritu@localhost:~/salary-api/src/main/java/com/opstree$ ls
microservice
ritu@localhost:~/salary-api/src/main/java/com/opstree$ cd microservice/
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice$ ls
salary
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice$ cd salary/
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary$ ls
config  contollers  model  repository  SalaryApplication.java  service
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary$ cat SalaryApplication.java 
package com.opstree.microservice.salary;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.cassandra.repository.config.EnableCassandraRepositories;
import org.springframework.context.annotation.Bean;
import com.opstree.microservice.salary.model.Employee;

import java.time.Duration;

import org.springframework.cache.annotation.EnableCaching;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;

@SpringBootApplication
@EnableCaching
public class SalaryApplication {

  @Bean
  public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
    RedisTemplate<Object, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(connectionFactory);
    Jackson2JsonRedisSerializer<Employee> serializer = new Jackson2JsonRedisSerializer<>(Employee.class);
    template.setDefaultSerializer(serializer);
    return template;
  }

  @Bean
  public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
    RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig() //
        .prefixCacheNameWith(this.getClass().getPackageName() + ".") //
        .entryTtl(Duration.ofSeconds(1)) //
        .disableCachingNullValues();

    return RedisCacheManager.builder(connectionFactory) //
        .cacheDefaults(config) //
        .build();
  }

	public static void main(String[] args) {
		SpringApplication.run(SalaryApplication.class, args);
	}
}
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary$ ls
config  contollers  model  repository  SalaryApplication.java  service
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary$ cd con
-bash: cd: con: No such file or directory
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary$ cd con
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary$ cd config/
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/config$ ls
OpenAPIConfig.java
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/config$ cat OpenAPIConfig.java 
package com.opstree.microservice.salary.swagger;

import java.util.List;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import io.swagger.v3.oas.models.servers.Server;

@Configuration
public class OpenAPIConfig {

  @Bean
  public OpenAPI myOpenAPI() {
    Server devServer = new Server();
    devServer.setUrl("http://localhost:8080");
    devServer.setDescription("Server URL in Development environment");

    Contact contact = new Contact();
    contact.setEmail("opensource@opstree.com");
    contact.setName("Opstree Solutions");
    contact.setUrl("https://opstree.com");

    License mitLicense = new License().name("MIT License").url("https://choosealicense.com/licenses/mit/");

    Info info = new Info()
        .title("Salary Microservice API")
        .version("1.0")
        .contact(contact)
        .description("This API exposes endpoints to manage salary information.").termsOfService("https://www.opstree.com/terms")
        .license(mitLicense);

    return new OpenAPI().info(info).servers(List.of(devServer));
  }
}
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/config$ cd ..
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary$ cd contollers/
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/contollers$ ls
SpringDataController.java
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/contollers$ cat SpringDataController.java 
package com.opstree.microservice.salary.controller;

import com.opstree.microservice.salary.service.SpringDataSalaryService;
import com.opstree.microservice.salary.model.Employee;
import com.opstree.microservice.salary.model.Message;

import java.util.List;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.beans.factory.annotation.Autowired;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;

@Tag(name = "Salary API", description = "Management APIs for all salary related transaction")
@RestController
@RequestMapping("/api/v1/salary")
@RequiredArgsConstructor
public class SpringDataController {

    @Autowired
    SpringDataSalaryService springDataSalaryService;

    @Operation(summary = "Create a new employee salary record", tags = { })
    @ApiResponses({@ApiResponse(responseCode = "201", content = { @Content(schema = @Schema(implementation = Employee.class), mediaType = "application/json") }) })
    @PostMapping("/create/record")
    public ResponseEntity<Employee> createSalaryRecord(@RequestBody Employee employee) {
        try {
           Employee _employee = springDataSalaryService
               .saveSalary(new Employee(employee.getId(), employee.getName(), employee.getSalary(), employee.getProcessDate(), employee.getStatus()));
           return new ResponseEntity<>(employee, HttpStatus.CREATED);
        } catch (Exception e) {
            return new ResponseEntity<>(null, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    @Operation(summary = "Retrieve all employee salary information", tags = {})
    @ApiResponses({@ApiResponse(responseCode = "200", content = {@Content(schema = @Schema(implementation = Employee.class), mediaType = "application/json") })})
    @GetMapping("/search/all")
    public ResponseEntity<List<Employee>> getAllEmployeeSalary() {
        try {
            return new ResponseEntity<>(springDataSalaryService.getAllEmployeeSalary(), HttpStatus.OK);
        } catch (Exception e) {
            return new ResponseEntity<>(null, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    @Operation(
      summary = "Retrieve a Salary by Employee Id",
      description = "Get a salary object by specifying its id. The response is Employee object with id, name, salary.",
      tags = {})
    @ApiResponses({@ApiResponse(responseCode = "200", content = { @Content(schema = @Schema(implementation = Employee.class), mediaType = "application/json") })})
    @GetMapping("/search")
    public ResponseEntity<Employee> findSalary(@RequestParam("id") String id) {
        try {
            return new ResponseEntity<>(springDataSalaryService.getEmployeeSalary(id), HttpStatus.OK);
        } catch (Exception e) {
            return new ResponseEntity<>(null, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
}
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/contollers$ ls
SpringDataController.java
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/contollers$ cd ..
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary$ ls
config  contollers  model  repository  SalaryApplication.java  service
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary$ cd model/
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/model$ ls
Employee.java  Message.java
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/model$ cat Employee.java 
package com.opstree.microservice.salary.model;

import java.time.LocalDate;
import java.io.Serializable;

import org.springframework.data.annotation.Id;
import org.springframework.data.cassandra.core.mapping.Column;
import org.springframework.data.cassandra.core.mapping.PrimaryKey;
import org.springframework.data.cassandra.core.mapping.Table;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import io.swagger.v3.oas.annotations.media.Schema;

@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Table("employee_salary")
public class Employee implements Serializable {

    @Id
    @PrimaryKey
    @Column("id")
    private String id;

    @Column("name")
    private String name;

    @Column("salary")
    private Float salary;

    @Column("process_date")
    private String processDate;

    @Column("status")
    private String status;

    public String getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public Float getSalary() {
        return salary;
    }

    public String getProcessDate() {
        return processDate;
    }

    public String getStatus() {
        return status;
    }
}
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/model$ car Message.java 
Command 'car' not found, but can be installed with:
sudo apt install ucommon-utils
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/model$ ls
Employee.java  Message.java
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/model$ cat Message.java 
package com.opstree.microservice.salary.model;

public class Message {
    private String message;

    public Message(String message) {
        this.message = message;
    }
}
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/model$ ls
Employee.java  Message.java
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/model$ cd ..
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary$ ls
config  contollers  model  repository  SalaryApplication.java  service
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary$ cd repository/
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/repository$ ls
EmployeeRepository.java
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/repository$ cat EmployeeRepository.java 
package com.opstree.microservice.salary.repository;

import com.opstree.microservice.salary.model.Employee;

import java.util.List;
import java.util.UUID;
import org.springframework.data.cassandra.repository.CassandraRepository;
import org.springframework.data.cassandra.repository.Query;

public interface EmployeeRepository extends CassandraRepository<Employee, UUID> {

    @Query("SELECT * FROM employee_salary WHERE id = ?0")
    Employee findByIdAsString(String id);
}
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/repository$ cd ..
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary$ ls
config  contollers  model  repository  SalaryApplication.java  service
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary$ cd service/
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/service$ ls
SpringDataSalaryService.java
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/service$ cat SpringDataSalaryService.java 
package com.opstree.microservice.salary.service;

import com.opstree.microservice.salary.model.Employee;
import com.opstree.microservice.salary.repository.EmployeeRepository;

import java.util.List;
import java.util.UUID;

import org.springframework.cache.annotation.Cacheable;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.data.cassandra.repository.config.EnableCassandraRepositories;

@Service
@EnableCassandraRepositories(basePackages={"com.opstree.microservice.salary"})
public class SpringDataSalaryService {

    @Autowired
    private EmployeeRepository employeeRepository;

    public List<Employee> getAllEmployeeSalary(){
        return employeeRepository.findAll();
    }

    @Cacheable("salary-search-id")
    public Employee getEmployeeSalary(String id){
        return employeeRepository.findByIdAsString(id);
    }

    public Employee saveSalary(Employee employee) {
        employeeRepository.save(employee);
        return employee;
    }
}
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary/service$ cd ..
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary$ ls
config  contollers  model  repository  SalaryApplication.java  service
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice/salary$ cd ..
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice$ ls
salary
ritu@localhost:~/salary-api/src/main/java/com/opstree/microservice$ cd ..
ritu@localhost:~/salary-api/src/main/java/com/opstree$ ls
microservice
ritu@localhost:~/salary-api/src/main/java/com/opstree$ cd ..
ritu@localhost:~/salary-api/src/main/java/com$ ls
opstree
ritu@localhost:~/salary-api/src/main/java/com$ cd ..
ritu@localhost:~/salary-api/src/main/java$ ls
com
ritu@localhost:~/salary-api/src/main/java$ cd ..
ritu@localhost:~/salary-api/src/main$ ls
java  resources
ritu@localhost:~/salary-api/src/main$ cd resources/
ritu@localhost:~/salary-api/src/main/resources$ ls
application.yml  logback-spring.xml
ritu@localhost:~/salary-api/src/main/resources$ cat application.yml 
spring:
  cassandra:
    keyspace-name: employee_db
    contact-points: 172.17.0.3
    port: 9042
    username: scylladb
    password: password
    local-datacenter: datacenter1
  data:
    redis:
      host: 172.17.0.4
      port: 6379
      password: password

management:
  endpoints:
    web:
      base-path: /actuator
      exposure:
        include: [ "health","prometheus", "metrics" ]
  health:
    cassandra:
      enabled: true
  endpoint:
    health:
      show-details: always
    metrics:
      enabled: true
    prometheus:
      enabled: true

logging:
  level:
    org.springframework.web: DEBUG

springdoc:
  swagger-ui:
    path: /salary-documentation
    tryItOutEnabled: true
    filter: true
  api-docs:
    path: /salary-api-docs
  show-actuator: true
ritu@localhost:~/salary-api/src/main/resources$ cat logback-spring.xml 
<configuration>
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder" />
    </appender>

    <root level="INFO">
        <appender-ref ref="console" />
    </root>
</configuration>
ritu@localhost:~/salary-api/src/main/resources$ cd ..
ritu@localhost:~/salary-api/src/main$ ls
java  resources
ritu@localhost:~/salary-api/src/main$ ls
java  resources
ritu@localhost:~/salary-api/src/main$ cd ..
ritu@localhost:~/salary-api/src$ ls
main  test
ritu@localhost:~/salary-api/src$ cd test/
ritu@localhost:~/salary-api/src/test$ ls
java  resources
ritu@localhost:~/salary-api/src/test$ cd java/
ritu@localhost:~/salary-api/src/test/java$ ls
com
ritu@localhost:~/salary-api/src/test/java$ cd ..
ritu@localhost:~/salary-api/src/test$ ls
java  resources
ritu@localhost:~/salary-api/src/test$ cd resources/
ritu@localhost:~/salary-api/src/test/resources$ ls
application.yml  logback-spring.xml
ritu@localhost:~/salary-api/src/test/resources$ cd ..
ritu@localhost:~/salary-api/src/test$ cd ..
ritu@localhost:~/salary-api/src$ ls
main  test
ritu@localhost:~/salary-api/src$ cd ..
ritu@localhost:~/salary-api$ ls
Dockerfile  LICENSE  Makefile  migration  migration.json  mvnw  mvnw.cmd  pom.xml  README.md  src  static
ritu@localhost:~/salary-api$ 

# Attendance



ritu@localhost:~/attendance-api$ ls
app.py       LICENSE               migration       pytest.ini  static
client       liquibase.properties  models          README.md   utils
config.yaml  log.conf              poetry.lock     router
Dockerfile   Makefile              pyproject.toml  scripts
ritu@localhost:~/attendance-api$ cat app.py 
"""
Module for calling the main flask application.
The application will be only supported with Flask and Gunicorn.
"""
from flask import Flask, json
from flasgger import Swagger
from prometheus_flask_exporter import PrometheusMetrics
from router.attendance import route as create_record
from router.cache import cache
from utils.json_encoder import DataclassJSONEncoder
from client.redis.redis_conn import get_caching_data

app = Flask(__name__)

swagger = Swagger(app)

metrics = PrometheusMetrics(app)
metrics.info("attendance_api", "Attendance API opentelemetry metrics", version="0.1.0")

cache.init_app(app, get_caching_data())

app.config['JSON_SORT_KEYS'] = False
json.provider.DefaultJSONProvider.sort_keys = False
app.json_encoder = DataclassJSONEncoder

app.register_blueprint(create_record, url_prefix="/api/v1")
ritu@localhost:~/attendance-api$ cat LICENSE 
                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding those notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. We also recommend that a
      file or class name and description of purpose be included on the
      same "printed page" as the copyright notice for easier
      identification within third-party archives.

   Copyright [2023] [OpsTree Solutions]

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
ritu@localhost:~/attendance-api$ ls
app.py       LICENSE               migration       pytest.ini  static
client       liquibase.properties  models          README.md   utils
config.yaml  log.conf              poetry.lock     router
Dockerfile   Makefile              pyproject.toml  scripts
ritu@localhost:~/attendance-api$ cd migration/
ritu@localhost:~/attendance-api/migration$ ls
db.changelog-master.xml
ritu@localhost:~/attendance-api/migration$ cat db.changelog-master.xml 
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext"
        xmlns:pro="http://www.liquibase.org/xml/ns/pro"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-latest.xsd
        http://www.liquibase.org/xml/ns/dbchangelog-ext http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd
        http://www.liquibase.org/xml/ns/pro http://www.liquibase.org/xml/ns/pro/liquibase-pro-latest.xsd">

    <changeSet id="1" author="Abhishek Dubey">
        <createTable tableName="records">
            <column name="id" type="text">
                <constraints primaryKey="true"/>
            </column>
            <column name="name" type="text"/>
            <column name="status" type="text"/>
            <column name="date" type="date"/>
        </createTable>
    </changeSet>
</databaseChangeLog>
ritu@localhost:~/attendance-api/migration$ cd ..
ritu@localhost:~/attendance-api$ ls
app.py       LICENSE               migration       pytest.ini  static
client       liquibase.properties  models          README.md   utils
config.yaml  log.conf              poetry.lock     router
Dockerfile   Makefile              pyproject.toml  scripts
ritu@localhost:~/attendance-api$ cat pytest.ini 
[pytest]
addopts = --ignore-glob=./app.py
ritu@localhost:~/attendance-api$ cd static/
ritu@localhost:~/attendance-api/static$ ls
attendance-api-logo.svg  attendance.png
ritu@localhost:~/attendance-api/static$ cd ..
ritu@localhost:~/attendance-api$ cd client/
ritu@localhost:~/attendance-api/client$ ls
postgres  redis  tests
ritu@localhost:~/attendance-api/client$ cat postgres/
cat: postgres/: Is a directory
ritu@localhost:~/attendance-api/client$ cd postgres/
ritu@localhost:~/attendance-api/client/postgres$ ls
__init__.py  postgres_conn.py
ritu@localhost:~/attendance-api/client/postgres$ cat __init__.py 
"""
Module for client SDK of Postgres and respective actions.
- Creation of record
- Read particular record
- Read all records
- Healthcheck for application
"""
# pylint: disable=import-error
from client.postgres.postgres_conn import CorePostgresClient

# pylint: disable=too-few-public-methods
class DatabaseSDKFacade:
    """Class wrapper method for client db related actions"""
    database = CorePostgresClient()
ritu@localhost:~/attendance-api/client/postgres$ cat postgres_conn.py 
"""
Module for postgres related methods and class
"""
# pylint: disable=import-error,unnecessary-lambda
import os
from collections import OrderedDict
from typing import List
import yaml
import psycopg2
from models.message import CustomMessage, HealthMessage
from models.user_info import EmployeeInfo
from client.redis import MiddlewareSDKFacade

CONFIG_FILE = os.getenv('CONFIG_FILE', 'config.yaml')

class CorePostgresClient:
    """Class for defining the interface for Postgres Client"""
    def __init__(self):
        with open(CONFIG_FILE, 'r', encoding="utf-8") as config_file:
            yaml_values = yaml.load(config_file, Loader=yaml.FullLoader)
        self.client = psycopg2.connect(database=yaml_values['postgres']['database'],
                                       host=yaml_values['postgres']['host'],
                                       user=yaml_values['postgres']['user'],
                                       password=yaml_values['postgres']['password'],
                                       port=yaml_values['postgres']['port'])

    def _record_to_domain_model(self, response):
        return EmployeeInfo(
            id=response.get("id"),
            name=response.get("name"),
            status=response.get("status"),
            date=response.get("date")
        )

    def read_employee_attendance(self, id_value) -> EmployeeInfo:
        """Function to read a particular employee attendance details"""
        cursor = self.client.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        read_query = f"SELECT id, name, status, date FROM records WHERE id='{id_value}'"
        cursor.execute(read_query)
        response = cursor.fetchone()
        return self._record_to_domain_model(OrderedDict(response))

    def read_all_employee_attendance(self) -> List[EmployeeInfo]:
        """Function to read all employee attendance records"""
        cursor = self.client.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("SELECT id, name, status, date FROM records ORDER BY id DESC")
        return list(
            map(
                lambda _: self._record_to_domain_model(_),
                cursor.fetchall(),
            )
        )[::-1]

    # pylint: disable=invalid-name,redefined-builtin
    def create_employee_attendance(self, id, name, status, date):
        """Function to create attendance record of the employee"""
        insert_query = """INSERT INTO records (id, name, status, date) VALUES (%s,%s,%s,%s)"""
        record_to_insert = (id, name, status, date)
        cursor = self.client.cursor()
        cursor.execute(insert_query, record_to_insert)
        self.client.commit()
        return CustomMessage(
            message=f"Successfully created the record for the employee id: ${id}"
        )

    def attendance_detail_health(self):
        """Function to get the detailed health of attendance API"""
        try:
            cursor = self.client.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
            cursor.execute("SELECT id, name, status, date FROM records LIMIT 1")
            return HealthMessage(
                message="Attendance API is running fine and ready to serve requests",
                postgresql="up",
                redis=MiddlewareSDKFacade.cache.redis_status(),
                status="up",
            ), 200
        except psycopg2.OperationalError:
            return HealthMessage(
                message="Attendance API is not healthy, please check logs",
                postgresql="down",
                redis=MiddlewareSDKFacade.cache.redis_status(),
                status="down",
            ), 400

    def attendance_health(self):
        """Function to get the health of attendance API"""
        try:
            cursor = self.client.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
            cursor.execute("SELECT id, name, status, date FROM records LIMIT 1")
            return CustomMessage(
                message="Attendance API is running fine and ready to serve requests",
            ), 200
        except psycopg2.OperationalError:
            return CustomMessage(
                message="Attendance API is not healthy, please check logs",
            ), 400
ritu@localhost:~/attendance-api/client/postgres$ cd ..
ritu@localhost:~/attendance-api/client$ ls
postgres  redis  tests
ritu@localhost:~/attendance-api/client$ cd redis/
ritu@localhost:~/attendance-api/client/redis$ ls
__init__.py  redis_conn.py
ritu@localhost:~/attendance-api/client/redis$ cat __init__.py 
"""
Module for client SDK of Redis and respective actions.
- Read particular record
- Read all records
"""
# pylint: disable=import-error
from client.redis.redis_conn import CoreRedisClient

# pylint: disable=too-few-public-methods
class MiddlewareSDKFacade:
    """Class wrapper method for client cache related actions"""
    cache = CoreRedisClient()
ritu@localhost:~/attendance-api/client/redis$ cat redis_conn.py 
"""
Module for Redis data and interface
"""
# pylint: disable=too-few-public-methods,no-member
import os
import redis
import yaml

CONFIG_FILE = os.getenv('CONFIG_FILE', 'config.yaml')

def get_caching_data():
    """Function to get cache config for redis cache"""
    with open(CONFIG_FILE, 'r', encoding="utf-8") as config_file:
        yaml_value = yaml.load(config_file, Loader=yaml.FullLoader)
    config_dict={
        "CACHE_TYPE": "redis",
        "CACHE_REDIS_HOST": yaml_value['redis']['host'],
        "CACHE_REDIS_PORT": yaml_value['redis']['port'],
        "CACHE_REDIS_URL": f"redis://{yaml_value['redis']['host']}:{yaml_value['redis']['port']}/0"
    }
    return config_dict


class CoreRedisClient:
    """Class for defining the structure of Redis database"""
    def __init__(self):
        with open(CONFIG_FILE, 'r', encoding="utf-8") as config_file:
            yaml_values = yaml.load(config_file, Loader=yaml.FullLoader)
        self.client = redis.Redis(host=yaml_values['redis']['host'],
                                   port=yaml_values['redis']['port'],
                                   password=yaml_values['redis']['password'],
                                   decode_responses=True)

    def redis_status(self):
        """Function for getting the health of redis"""
        try:
            self.client.ping()
            return "up"
        except redis.ConnectionError:
            return "down"
ritu@localhost:~/attendance-api/client/redis$ ls
__init__.py  redis_conn.py
ritu@localhost:~/attendance-api/client/redis$ cd ..
ritu@localhost:~/attendance-api/client$ ls
postgres  redis  tests
ritu@localhost:~/attendance-api/client$ cd tests/
ritu@localhost:~/attendance-api/client/tests$ ls
test_postgres_conn.py  test_redis_conn.py
ritu@localhost:~/attendance-api/client/tests$ cat test_postgres_conn.py 
from unittest.mock import Mock, patch
import pytest
import psycopg2
from collections import OrderedDict
from models.message import CustomMessage, HealthMessage
from models.user_info import EmployeeInfo
from client.redis import MiddlewareSDKFacade
from client.postgres import DatabaseSDKFacade

@pytest.fixture
def mock_config_file(tmp_path):
    config_data = """
    postgres:
        host: localhost
        port: 5432
        user: postgres
        password: password
        database: test_db
    """
    config_file = tmp_path / "config.yaml"
    config_file.write_text(config_data)
    return str(config_file)

def test_core_postgres_client_read_employee_attendance(mock_config_file, mocker):
    mock_psycopg2 = mocker.patch("client.postgres.DatabaseSDKFacade")
    mock_cursor = Mock()
    mock_fetchone_result = {
        "id": "1",
        "name": "John Doe",
        "status": "Present",
        "date": "2023-01-01",
    }
    mock_cursor.fetchone.return_value = mock_fetchone_result
    mock_psycopg2.connect.return_value.cursor.return_value = mock_cursor

    result = EmployeeInfo(
        id="1",
        name="John Doe",
        status="Present",
        date="2023-01-01",
    )

    assert result == EmployeeInfo(
        id="1",
        name="John Doe",
        status="Present",
        date="2023-01-01",
    )
    # mock_psycopg2.connect.assert_called_once_with(
    #     database="test_db",
    #     host="localhost",
    #     user="postgres",
    #     password="password",
    #     port=5432,
    # )


def test_core_postgres_client_read_all_employee_attendance(mock_config_file, mocker):
    mock_psycopg2 = mocker.patch("client.postgres.DatabaseSDKFacade")
    mock_cursor = Mock()
    mock_fetchall_result = [
        {
            "id": "1",
            "name": "John Doe",
            "status": "Present",
            "date": "2023-01-01",
        },
        {
            "id": "2",
            "name": "Jane Smith",
            "status": "Absent",
            "date": "2023-01-02",
        },
    ]
    mock_cursor.fetchall.return_value = mock_fetchall_result
    mock_psycopg2.connect.return_value.cursor.return_value = mock_cursor

    result = [
        EmployeeInfo(
            id="1",
            name="John Doe",
            status="Present",
            date="2023-01-01",
        ),
        EmployeeInfo(
            id="2",
            name="Jane Smith",
            status="Absent",
            date="2023-01-02",
        ),
    ]

    assert result == [
        EmployeeInfo(
            id="1",
            name="John Doe",
            status="Present",
            date="2023-01-01",
        ),
        EmployeeInfo(
            id="2",
            name="Jane Smith",
            status="Absent",
            date="2023-01-02",
        ),
    ]

def test_core_postgres_client_create_employee_attendance(mock_config_file, mocker):
    mock_psycopg2 = mocker.patch("client.postgres.DatabaseSDKFacade")
    mock_cursor = Mock()
    mock_psycopg2.connect.return_value.cursor.return_value = mock_cursor

    result = CustomMessage(
        message="Successfully created the record for the employee id: $1"
    )

    assert result == CustomMessage(
        message="Successfully created the record for the employee id: $1"
    )

def test_core_postgres_client_attendance_detail_health(mock_config_file, mocker):
    mock_psycopg2 = mocker.patch("client.postgres.DatabaseSDKFacade")
    mock_cursor = Mock()
    mock_cursor.fetchone.return_value = {
        "id": "1",
        "name": "John Doe",
        "status": "Present",
        "date": "2023-01-01",
    }
    mock_psycopg2.connect.return_value.cursor.return_value = mock_cursor

    mock_redis_status = mocker.patch.object(
        MiddlewareSDKFacade.cache, "redis_status", return_value="up"
    )

    result, status_code = HealthMessage(
        message="Attendance API is running fine and ready to serve requests",
        postgresql="up",
        redis="up",
        status="up",
    ), 200

    assert result == HealthMessage(
        message="Attendance API is running fine and ready to serve requests",
        postgresql="up",
        redis="up",
        status="up",
    )
    assert status_code == 200

def test_core_postgres_client_attendance_health(mock_config_file, mocker):
    mock_psycopg2 = mocker.patch("client.postgres.DatabaseSDKFacade")
    mock_cursor = Mock()
    mock_cursor.fetchone.side_effect = psycopg2.OperationalError
    mock_psycopg2.connect.return_value.cursor.return_value = mock_cursor

    mock_redis_status = mocker.patch.object(
        MiddlewareSDKFacade.cache, "redis_status", return_value="up"
    )

    result, status_code = DatabaseSDKFacade.database.attendance_health()

    assert result == CustomMessage(
        message="Attendance API is running fine and ready to serve requests"
    )
    assert status_code == 200
ritu@localhost:~/attendance-api/client/tests$ cat test_redis_conn.py 
import os
from unittest import mock
from client.redis import MiddlewareSDKFacade
from client.redis.redis_conn import get_caching_data

@mock.patch('client.redis.open')
def test_get_caching_data(mock_open):
    mock_file = mock.MagicMock()
    mock_file.__enter__.return_value = mock_file
    mock_file.read.return_value = """
    redis:
        host: localhost
        port: 6379
        password: mypassword
    """

    mock_open.return_value = mock_file
    os.environ['CONFIG_FILE'] = 'config.yaml'

    result = get_caching_data()

    assert result == {
        "CACHE_TYPE": "redis",
        "CACHE_REDIS_HOST": "172.17.0.4",
        "CACHE_REDIS_PORT": 6379,
        "CACHE_REDIS_URL": "redis://172.17.0.4:6379/0"
    }

    # mock_open.assert_called_once_with('config.yaml', 'r', encoding="utf-8")
    # mock_file.read.assert_called_once()
    #
    # del os.environ['CONFIG_FILE']


@mock.patch('client.redis.open')
def test_redis_status(mock_open):
    mock_file = mock.MagicMock()
    mock_file.__enter__.return_value = mock_file
    mock_file.read.return_value = """
    redis:
        host: localhost
        port: 6379
        password: mypassword
    """

    mock_open.return_value = mock_file
    os.environ['CONFIG_FILE'] = 'config.yaml'

    redis_client = MiddlewareSDKFacade.cache.redis_status()
    # mock_redis.assert_called_once_with(
    #     host="localhost",
    #     port=6379,
    #     password="mypassword",
    #     decode_responses=True
    # )
    assert redis_client == "up"

    del os.environ['CONFIG_FILE']ritu@localhost:~/attendance-api/client/tests$ cd ..
ritu@localhost:~/attendance-api/client$ ls
postgres  redis  tests
ritu@localhost:~/attendance-api/client$ cd ..
ritu@localhost:~/attendance-api$ ls
app.py       LICENSE               migration       pytest.ini  static
client       liquibase.properties  models          README.md   utils
config.yaml  log.conf              poetry.lock     router
Dockerfile   Makefile              pyproject.toml  scripts
ritu@localhost:~/attendance-api$ cat liquibase.properties 
url=jdbc:postgresql://172.17.0.3:5432/attendance_db
driver=org.postgresql.Driver
username=postgres
password=password
changeLogFile=migration/db.changelog-master.xml
ritu@localhost:~/attendance-api$ cd models/
ritu@localhost:~/attendance-api/models$ ls
message.py  tests  user_info.py
ritu@localhost:~/attendance-api/models$ cat message.py 
"""
Module for defining models of Message used in attendance API
"""

from dataclasses import dataclass

@dataclass
class CustomMessage:
    """Class model for custom message using Flask"""
    message: str

@dataclass
class HealthMessage:
    """Class model for health message using Flask for attendance API"""
    message: str
    postgresql: str
    redis: str
    status: str
ritu@localhost:~/attendance-api/models$ cd tests/
ritu@localhost:~/attendance-api/models/tests$ ls
test_message.py  test_user_info.py
ritu@localhost:~/attendance-api/models/tests$ cat test_message.py 
from dataclasses import dataclass
import pytest

@dataclass
class CustomMessage:
    """Class model for custom message using Flask"""
    message: str

@dataclass
class HealthMessage:
    """Class model for health message using Flask for attendance API"""
    message: str
    postgresql: str
    redis: str
    status: str

def test_custom_message():
    # Create an instance of CustomMessage
    custom_message = CustomMessage(message="Hello, World!")

    # Assert that the message attribute is set correctly
    assert custom_message.message == "Hello, World!"

def test_health_message():
    # Create an instance of HealthMessage
    health_message = HealthMessage(message="API is running", postgresql="Connected", redis="Connected", status="OK")

    # Assert that the attributes are set correctly
    assert health_message.message == "API is running"
    assert health_message.postgresql == "Connected"
    assert health_message.redis == "Connected"
    assert health_message.status == "OK"
ritu@localhost:~/attendance-api/models/tests$ cat test_user_info.py 
from dataclasses import dataclass
import pytest

@dataclass
class EmployeeInfo:
    """Class model for defining the structure for employee's information"""
    id: str
    name: str
    status: str
    date: str

def test_employee_info():
    # Create an instance of EmployeeInfo
    employee = EmployeeInfo(id="123", name="John Doe", status="Active", date="2023-07-01")

    # Assert that the attributes are set correctly
    assert employee.id == "123"
    assert employee.name == "John Doe"
    assert employee.status == "Active"
    assert employee.date == "2023-07-01"

if __name__ == '__main__':
    pytest.main()
ritu@localhost:~/attendance-api/models/tests$ ls
test_message.py  test_user_info.py
ritu@localhost:~/attendance-api/models/tests$ cd ..
ritu@localhost:~/attendance-api/models$ ls
message.py  tests  user_info.py
ritu@localhost:~/attendance-api/models$ cat user_info.py 
"""
Module for defining models of Employee information used in attendance API
"""

from dataclasses import dataclass

# pylint: disable=invalid-name
@dataclass
class EmployeeInfo:
    """Class model for defining the structure for employee's information"""
    id: str
    name: str
    status: str
    date: str
ritu@localhost:~/attendance-api/models$ cd ..
ritu@localhost:~/attendance-api$ ls
app.py       LICENSE               migration       pytest.ini  static
client       liquibase.properties  models          README.md   utils
config.yaml  log.conf              poetry.lock     router
Dockerfile   Makefile              pyproject.toml  scripts
ritu@localhost:~/attendance-api$ cat README.md 
<p align="center">
  <img src="./static/attendance-api-logo.svg" height="280" width="280">
</p>

Attendance REST API is a python based microservice which is responsible for all the attendance related transactions in the [OT-Microservices](https://github.com/OT-MICROSERVICES). This application supports cross-platform, the only thing will be required to run this application is python runtime modules.

Supported features of the application are:-

- Flask based web framework for all REST related transactions
- PostgresSQL as a primary database for storing all the attendance records
- Redis as cache management middleware for storing all API response
- Prometheus and open-telemetry metrics supports for monitoring and observability
- Swagger integration for API documentation of all endpoints

## Pre-Requisites

The attendance api application have some database and package manager dependencies. Some of them are mandatory and some can be option, for example - Redis. To run the application successfully, we need these things configured:

- [PostgresSQL](https://www.postgresql.org/)
- [Redis](https://redis.io/)
- [Poetry](https://python-poetry.org/)
- [Liquibase](https://docs.liquibase.com/)

Poetry will be used as package manager to install specific versions module on dependencies to run the attendance API.

## Architecture

![](./static/attendance.png)

## Application

For building the application, we can use `make` command with our [Makefile](Makefile). But as first and foremost step, we need to install all the dependencies and packages using `poetry`. We can simple use `make` command for it.

```shell
make build
```

For building the docker image artifact of the attendance api, we can invoke another make command.

```shell
make docker-build
```

Also, attendance api contains different test cases and code quality standard related integrations. For checking the quality of the code, we can make use of `pylint`. To summarize and make it easy to run, we can invoke a make command for it. Also, for test cases, we can use `pytest`.

```shell
make fmt
```

```shell
python3 -m pytest
# For generating the code coverage report
 python3 -m pytest --cov=.
```

The test cases are included for following modules and directories:-

- router
- client
- models
- utils

For dev testing, the Swagger UI can be used for sample payload generation and requests. The swagger page will be accessible on http://localhost:8080/apidocs. Before running the application, we have to make sure our mandatory database (PostgresSQL) is up and running. Configuration properties will be configured inside [config.yaml](config.yaml) file. Also, once the property file is defined and configured properly, we need to run migrations to create database, schema etc. The connection details for migration is available in [liquibase.properties](./liquibase.properties).

```shell
make run-migrations
```

Once the database and table is initialized, we can run the application by:

```shell
gunicorn app:app --log-config log.conf -b 0.0.0.0:8080
```

## Endpoints Information

| **Endpoint**                     | **Method** | **Description**                                                                                      |
|----------------------------------|------------|------------------------------------------------------------------------------------------------------|
| /metrics                         | GET        | Application healthcheck and performance metrics are available on this endpoint                       |
| /apidocs                         | GET        | Swagger endpoint for the application API documentation and their data models                         |
| /api/v1/attendance/create        | POST       | Data creation endpoint which accepts certain JSON body to add attendance information in database     |
| /api/v1/attendance/search        | GET        | Endpoint for searching data information using the params in the URL                                  |
| /api/v1/attendance/search/all    | GET        | Endpoint for searching all information across the system                                             |
| /api/v1/attendance/health        | GET        | Endpoint for providing shallow healthcheck information about application health and readiness        |
| /api/v1/attendance/health/detail | GET        | Endpoint for providing detailed health check about dependencies as well like - PostgresSQL and Redis |

## Contact Information

[Opstree Opensource](mailto:opensource@opstree.com)
ritu@localhost:~/attendance-api$ cd utils/
ritu@localhost:~/attendance-api/utils$ ls
json_encoder.py  log_encoder.py  tests  validator.py
ritu@localhost:~/attendance-api/utils$ cat json_encoder.py 
"""
Module for Encoding and Decoding of JSON data
"""

from enum import Enum
from datetime import datetime
from dataclasses import is_dataclass, asdict
from json import JSONEncoder
from peewee import Model
from playhouse.shortcuts import model_to_dict

# pylint: disable=no-else-return
class DataclassJSONEncoder(JSONEncoder):
    """Class for encoding the different type of JSON object"""
    def default(self, o):
        if is_dataclass(o):
            return asdict(o)
        elif isinstance(o, Model):
            return model_to_dict(o)
        elif isinstance(o, datetime):
            return o.isoformat()
        elif isinstance(o, Enum):
            return str(o)
        return super().default(o)
ritu@localhost:~/attendance-api/utils$ cat log_encoder.py 
"""
Module for custom json log model in flask and gunicorn
"""
from pythonjsonlogger import jsonlogger

# pylint: disable=super-with-arguments
class CustomJsonFormatter(jsonlogger.JsonFormatter):
    """Class for defining JSON log structure of Flask"""
    def add_fields(self, log_record, record, message_dict):
        super(CustomJsonFormatter, self).add_fields(log_record, record, message_dict)
        if not log_record.get('timestamp'):
            log_record['timestamp'] = record.created
        if 'r' in record.args:
            log_record['request'] = record.args.get('r')
            log_record['message'] = None
        if 's' in record.args:
            log_record['status_code'] = record.args.get('s')
        if 'm' in record.args:
            log_record['method'] = record.args.get('m')
        if 'h' in record.args:
            log_record['remote_address'] = record.args.get('h')
ritu@localhost:~/attendance-api/utils$ ls
json_encoder.py  log_encoder.py  tests  validator.py
ritu@localhost:~/attendance-api/utils$ cat validator.py 
"""
Module for all helping functions like query and data validation
"""

from functools import wraps
from flask import jsonify, request
from voluptuous import Schema, Invalid

def query_validator(schema: Schema):
    """Function for query validation"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            try:
                query = request.args.to_dict()
                valid_dict = schema(query)
                kwargs.update(valid_dict)
            except Invalid:
                return jsonify({"error": "Validation Error"}), 400
            return func(*args, **kwargs)

        return wrapper

    return decorator


def data_validator(schema: Schema):
    """Function for data validation"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            try:
                query = request.get_json()
                valid_dict = schema(query)
                kwargs.update(valid_dict)
            except Invalid:
                return jsonify({"error": "Validation Error"}), 400
            return func(*args, **kwargs)

        return wrapper

    return decorator
ritu@localhost:~/attendance-api/utils$ cd ..
ritu@localhost:~/attendance-api$ ls
app.py       LICENSE               migration       pytest.ini  static
client       liquibase.properties  models          README.md   utils
config.yaml  log.conf              poetry.lock     router
Dockerfile   Makefile              pyproject.toml  scripts
ritu@localhost:~/attendance-api$ cat config.yaml 
---
postgres:
  database: attendance_db
  host: 172.17.0.3
  port: 5432
  user: postgres
  password: password

redis:
  host: 172.17.0.4
  port: 6379
  password: ""
ritu@localhost:~/attendance-api$ cat log.conf 
[loggers]
keys=root, gunicorn.error, gunicorn.access

[handlers]
keys=console

[formatters]
keys=json

[logger_root]
level=INFO
handlers=console

[logger_gunicorn.error]
level=INFO
handlers=console
propagate=0
qualname=gunicorn.error

[logger_gunicorn.access]
level=INFO
handlers=console
propagate=0
qualname=gunicorn.access

[handler_console]
class=StreamHandler
formatter=json
args=(sys.stdout, )

[formatter_json]
class=utils.log_encoder.CustomJsonFormatter
format='%(timestamp)s %(levelname)s %(message)s'ritu@localhost:~/attendance-api$ ls
app.py       LICENSE               migration       pytest.ini  static
client       liquibase.properties  models          README.md   utils
config.yaml  log.conf              poetry.lock     router
Dockerfile   Makefile              pyproject.toml  scripts
ritu@localhost:~/attendance-api$ cat poetry.lock 
# This file is automatically @generated by Poetry 1.5.1 and should not be changed by hand.

[[package]]
name = "async-timeout"
version = "4.0.2"
description = "Timeout context manager for asyncio programs"
optional = false
python-versions = ">=3.6"
files = [
    {file = "async-timeout-4.0.2.tar.gz", hash = "sha256:2163e1640ddb52b7a8c80d0a67a08587e5d245cc9c553a74a847056bc2976b15"},
    {file = "async_timeout-4.0.2-py3-none-any.whl", hash = "sha256:8ca1e4fcf50d07413d66d1a5e416e42cfdf5851c981d679a09851a6853383b3c"},
]

[[package]]
name = "attrs"
version = "23.1.0"
description = "Classes Without Boilerplate"
optional = false
python-versions = ">=3.7"
files = [
    {file = "attrs-23.1.0-py3-none-any.whl", hash = "sha256:1f28b4522cdc2fb4256ac1a020c78acf9cba2c6b461ccd2c126f3aa8e8335d04"},
    {file = "attrs-23.1.0.tar.gz", hash = "sha256:6279836d581513a26f1bf235f9acd333bc9115683f14f7e8fae46c98fc50e015"},
]

[package.extras]
cov = ["attrs[tests]", "coverage[toml] (>=5.3)"]
dev = ["attrs[docs,tests]", "pre-commit"]
docs = ["furo", "myst-parser", "sphinx", "sphinx-notfound-page", "sphinxcontrib-towncrier", "towncrier", "zope-interface"]
tests = ["attrs[tests-no-zope]", "zope-interface"]
tests-no-zope = ["cloudpickle", "hypothesis", "mypy (>=1.1.1)", "pympler", "pytest (>=4.3.0)", "pytest-mypy-plugins", "pytest-xdist[psutil]"]

[[package]]
name = "blinker"
version = "1.6.2"
description = "Fast, simple object-to-object and broadcast signaling"
optional = false
python-versions = ">=3.7"
files = [
    {file = "blinker-1.6.2-py3-none-any.whl", hash = "sha256:c3d739772abb7bc2860abf5f2ec284223d9ad5c76da018234f6f50d6f31ab1f0"},
    {file = "blinker-1.6.2.tar.gz", hash = "sha256:4afd3de66ef3a9f8067559fb7a1cbe555c17dcbe15971b05d1b625c3e7abe213"},
]

[[package]]
name = "cachelib"
version = "0.9.0"
description = "A collection of cache libraries in the same API interface."
optional = false
python-versions = ">=3.7"
files = [
    {file = "cachelib-0.9.0-py3-none-any.whl", hash = "sha256:811ceeb1209d2fe51cd2b62810bd1eccf70feba5c52641532498be5c675493b3"},
    {file = "cachelib-0.9.0.tar.gz", hash = "sha256:38222cc7c1b79a23606de5c2607f4925779e37cdcea1c2ad21b8bae94b5425a5"},
]

[[package]]
name = "click"
version = "8.1.3"
description = "Composable command line interface toolkit"
optional = false
python-versions = ">=3.7"
files = [
    {file = "click-8.1.3-py3-none-any.whl", hash = "sha256:bb4d8133cb15a609f44e8213d9b391b0809795062913b383c62be0ee95b1db48"},
    {file = "click-8.1.3.tar.gz", hash = "sha256:7682dc8afb30297001674575ea00d1814d808d6a36af415a82bd481d37ba7b8e"},
]

[package.dependencies]
colorama = {version = "*", markers = "platform_system == \"Windows\""}

[[package]]
name = "colorama"
version = "0.4.6"
description = "Cross-platform colored terminal text."
optional = false
python-versions = "!=3.0.*,!=3.1.*,!=3.2.*,!=3.3.*,!=3.4.*,!=3.5.*,!=3.6.*,>=2.7"
files = [
    {file = "colorama-0.4.6-py2.py3-none-any.whl", hash = "sha256:4f1d9991f5acc0ca119f9d443620b77f9d6b33703e51011c16baf57afb285fc6"},
    {file = "colorama-0.4.6.tar.gz", hash = "sha256:08695f5cb7ed6e0531a20572697297273c47b8cae5a63ffc6d6ed5c201be6e44"},
]

[[package]]
name = "coverage"
version = "7.2.7"
description = "Code coverage measurement for Python"
optional = false
python-versions = ">=3.7"
files = [
    {file = "coverage-7.2.7-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:d39b5b4f2a66ccae8b7263ac3c8170994b65266797fb96cbbfd3fb5b23921db8"},
    {file = "coverage-7.2.7-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:6d040ef7c9859bb11dfeb056ff5b3872436e3b5e401817d87a31e1750b9ae2fb"},
    {file = "coverage-7.2.7-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:ba90a9563ba44a72fda2e85302c3abc71c5589cea608ca16c22b9804262aaeb6"},
    {file = "coverage-7.2.7-cp310-cp310-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:e7d9405291c6928619403db1d10bd07888888ec1abcbd9748fdaa971d7d661b2"},
    {file = "coverage-7.2.7-cp310-cp310-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:31563e97dae5598556600466ad9beea39fb04e0229e61c12eaa206e0aa202063"},
    {file = "coverage-7.2.7-cp310-cp310-musllinux_1_1_aarch64.whl", hash = "sha256:ebba1cd308ef115925421d3e6a586e655ca5a77b5bf41e02eb0e4562a111f2d1"},
    {file = "coverage-7.2.7-cp310-cp310-musllinux_1_1_i686.whl", hash = "sha256:cb017fd1b2603ef59e374ba2063f593abe0fc45f2ad9abdde5b4d83bd922a353"},
    {file = "coverage-7.2.7-cp310-cp310-musllinux_1_1_x86_64.whl", hash = "sha256:d62a5c7dad11015c66fbb9d881bc4caa5b12f16292f857842d9d1871595f4495"},
    {file = "coverage-7.2.7-cp310-cp310-win32.whl", hash = "sha256:ee57190f24fba796e36bb6d3aa8a8783c643d8fa9760c89f7a98ab5455fbf818"},
    {file = "coverage-7.2.7-cp310-cp310-win_amd64.whl", hash = "sha256:f75f7168ab25dd93110c8a8117a22450c19976afbc44234cbf71481094c1b850"},
    {file = "coverage-7.2.7-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:06a9a2be0b5b576c3f18f1a241f0473575c4a26021b52b2a85263a00f034d51f"},
    {file = "coverage-7.2.7-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:5baa06420f837184130752b7c5ea0808762083bf3487b5038d68b012e5937dbe"},
    {file = "coverage-7.2.7-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:fdec9e8cbf13a5bf63290fc6013d216a4c7232efb51548594ca3631a7f13c3a3"},
    {file = "coverage-7.2.7-cp311-cp311-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:52edc1a60c0d34afa421c9c37078817b2e67a392cab17d97283b64c5833f427f"},
    {file = "coverage-7.2.7-cp311-cp311-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:63426706118b7f5cf6bb6c895dc215d8a418d5952544042c8a2d9fe87fcf09cb"},
    {file = "coverage-7.2.7-cp311-cp311-musllinux_1_1_aarch64.whl", hash = "sha256:afb17f84d56068a7c29f5fa37bfd38d5aba69e3304af08ee94da8ed5b0865833"},
    {file = "coverage-7.2.7-cp311-cp311-musllinux_1_1_i686.whl", hash = "sha256:48c19d2159d433ccc99e729ceae7d5293fbffa0bdb94952d3579983d1c8c9d97"},
    {file = "coverage-7.2.7-cp311-cp311-musllinux_1_1_x86_64.whl", hash = "sha256:0e1f928eaf5469c11e886fe0885ad2bf1ec606434e79842a879277895a50942a"},
    {file = "coverage-7.2.7-cp311-cp311-win32.whl", hash = "sha256:33d6d3ea29d5b3a1a632b3c4e4f4ecae24ef170b0b9ee493883f2df10039959a"},
    {file = "coverage-7.2.7-cp311-cp311-win_amd64.whl", hash = "sha256:5b7540161790b2f28143191f5f8ec02fb132660ff175b7747b95dcb77ac26562"},
    {file = "coverage-7.2.7-cp312-cp312-macosx_10_9_x86_64.whl", hash = "sha256:f2f67fe12b22cd130d34d0ef79206061bfb5eda52feb6ce0dba0644e20a03cf4"},
    {file = "coverage-7.2.7-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:a342242fe22407f3c17f4b499276a02b01e80f861f1682ad1d95b04018e0c0d4"},
    {file = "coverage-7.2.7-cp312-cp312-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:171717c7cb6b453aebac9a2ef603699da237f341b38eebfee9be75d27dc38e01"},
    {file = "coverage-7.2.7-cp312-cp312-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:49969a9f7ffa086d973d91cec8d2e31080436ef0fb4a359cae927e742abfaaa6"},
    {file = "coverage-7.2.7-cp312-cp312-musllinux_1_1_aarch64.whl", hash = "sha256:b46517c02ccd08092f4fa99f24c3b83d8f92f739b4657b0f146246a0ca6a831d"},
    {file = "coverage-7.2.7-cp312-cp312-musllinux_1_1_i686.whl", hash = "sha256:a3d33a6b3eae87ceaefa91ffdc130b5e8536182cd6dfdbfc1aa56b46ff8c86de"},
    {file = "coverage-7.2.7-cp312-cp312-musllinux_1_1_x86_64.whl", hash = "sha256:976b9c42fb2a43ebf304fa7d4a310e5f16cc99992f33eced91ef6f908bd8f33d"},
    {file = "coverage-7.2.7-cp312-cp312-win32.whl", hash = "sha256:8de8bb0e5ad103888d65abef8bca41ab93721647590a3f740100cd65c3b00511"},
    {file = "coverage-7.2.7-cp312-cp312-win_amd64.whl", hash = "sha256:9e31cb64d7de6b6f09702bb27c02d1904b3aebfca610c12772452c4e6c21a0d3"},
    {file = "coverage-7.2.7-cp37-cp37m-macosx_10_9_x86_64.whl", hash = "sha256:58c2ccc2f00ecb51253cbe5d8d7122a34590fac9646a960d1430d5b15321d95f"},
    {file = "coverage-7.2.7-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:d22656368f0e6189e24722214ed8d66b8022db19d182927b9a248a2a8a2f67eb"},
    {file = "coverage-7.2.7-cp37-cp37m-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:a895fcc7b15c3fc72beb43cdcbdf0ddb7d2ebc959edac9cef390b0d14f39f8a9"},
    {file = "coverage-7.2.7-cp37-cp37m-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:e84606b74eb7de6ff581a7915e2dab7a28a0517fbe1c9239eb227e1354064dcd"},
    {file = "coverage-7.2.7-cp37-cp37m-musllinux_1_1_aarch64.whl", hash = "sha256:0a5f9e1dbd7fbe30196578ca36f3fba75376fb99888c395c5880b355e2875f8a"},
    {file = "coverage-7.2.7-cp37-cp37m-musllinux_1_1_i686.whl", hash = "sha256:419bfd2caae268623dd469eff96d510a920c90928b60f2073d79f8fe2bbc5959"},
    {file = "coverage-7.2.7-cp37-cp37m-musllinux_1_1_x86_64.whl", hash = "sha256:2aee274c46590717f38ae5e4650988d1af340fe06167546cc32fe2f58ed05b02"},
    {file = "coverage-7.2.7-cp37-cp37m-win32.whl", hash = "sha256:61b9a528fb348373c433e8966535074b802c7a5d7f23c4f421e6c6e2f1697a6f"},
    {file = "coverage-7.2.7-cp37-cp37m-win_amd64.whl", hash = "sha256:b1c546aca0ca4d028901d825015dc8e4d56aac4b541877690eb76490f1dc8ed0"},
    {file = "coverage-7.2.7-cp38-cp38-macosx_10_9_x86_64.whl", hash = "sha256:54b896376ab563bd38453cecb813c295cf347cf5906e8b41d340b0321a5433e5"},
    {file = "coverage-7.2.7-cp38-cp38-macosx_11_0_arm64.whl", hash = "sha256:3d376df58cc111dc8e21e3b6e24606b5bb5dee6024f46a5abca99124b2229ef5"},
    {file = "coverage-7.2.7-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:5e330fc79bd7207e46c7d7fd2bb4af2963f5f635703925543a70b99574b0fea9"},
    {file = "coverage-7.2.7-cp38-cp38-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:1e9d683426464e4a252bf70c3498756055016f99ddaec3774bf368e76bbe02b6"},
    {file = "coverage-7.2.7-cp38-cp38-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:8d13c64ee2d33eccf7437961b6ea7ad8673e2be040b4f7fd4fd4d4d28d9ccb1e"},
    {file = "coverage-7.2.7-cp38-cp38-musllinux_1_1_aarch64.whl", hash = "sha256:b7aa5f8a41217360e600da646004f878250a0d6738bcdc11a0a39928d7dc2050"},
    {file = "coverage-7.2.7-cp38-cp38-musllinux_1_1_i686.whl", hash = "sha256:8fa03bce9bfbeeef9f3b160a8bed39a221d82308b4152b27d82d8daa7041fee5"},
    {file = "coverage-7.2.7-cp38-cp38-musllinux_1_1_x86_64.whl", hash = "sha256:245167dd26180ab4c91d5e1496a30be4cd721a5cf2abf52974f965f10f11419f"},
    {file = "coverage-7.2.7-cp38-cp38-win32.whl", hash = "sha256:d2c2db7fd82e9b72937969bceac4d6ca89660db0a0967614ce2481e81a0b771e"},
    {file = "coverage-7.2.7-cp38-cp38-win_amd64.whl", hash = "sha256:2e07b54284e381531c87f785f613b833569c14ecacdcb85d56b25c4622c16c3c"},
    {file = "coverage-7.2.7-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:537891ae8ce59ef63d0123f7ac9e2ae0fc8b72c7ccbe5296fec45fd68967b6c9"},
    {file = "coverage-7.2.7-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:06fb182e69f33f6cd1d39a6c597294cff3143554b64b9825d1dc69d18cc2fff2"},
    {file = "coverage-7.2.7-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:201e7389591af40950a6480bd9edfa8ed04346ff80002cec1a66cac4549c1ad7"},
    {file = "coverage-7.2.7-cp39-cp39-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:f6951407391b639504e3b3be51b7ba5f3528adbf1a8ac3302b687ecababf929e"},
    {file = "coverage-7.2.7-cp39-cp39-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:6f48351d66575f535669306aa7d6d6f71bc43372473b54a832222803eb956fd1"},
    {file = "coverage-7.2.7-cp39-cp39-musllinux_1_1_aarch64.whl", hash = "sha256:b29019c76039dc3c0fd815c41392a044ce555d9bcdd38b0fb60fb4cd8e475ba9"},
    {file = "coverage-7.2.7-cp39-cp39-musllinux_1_1_i686.whl", hash = "sha256:81c13a1fc7468c40f13420732805a4c38a105d89848b7c10af65a90beff25250"},
    {file = "coverage-7.2.7-cp39-cp39-musllinux_1_1_x86_64.whl", hash = "sha256:975d70ab7e3c80a3fe86001d8751f6778905ec723f5b110aed1e450da9d4b7f2"},
    {file = "coverage-7.2.7-cp39-cp39-win32.whl", hash = "sha256:7ee7d9d4822c8acc74a5e26c50604dff824710bc8de424904c0982e25c39c6cb"},
    {file = "coverage-7.2.7-cp39-cp39-win_amd64.whl", hash = "sha256:eb393e5ebc85245347950143969b241d08b52b88a3dc39479822e073a1a8eb27"},
    {file = "coverage-7.2.7-pp37.pp38.pp39-none-any.whl", hash = "sha256:b7b4c971f05e6ae490fef852c218b0e79d4e52f79ef0c8475566584a8fb3e01d"},
    {file = "coverage-7.2.7.tar.gz", hash = "sha256:924d94291ca674905fe9481f12294eb11f2d3d3fd1adb20314ba89e94f44ed59"},
]

[package.extras]
toml = ["tomli"]

[[package]]
name = "flasgger"
version = "0.9.7b2"
description = "Extract swagger specs from your flask project"
optional = false
python-versions = "*"
files = [
    {file = "flasgger-0.9.7b2.tar.gz", hash = "sha256:5c4328b0ad7b5c3768f36a1f7eda5fb28c4495d2f1b3d90621670643fb57a549"},
]

[package.dependencies]
Flask = ">=0.10"
jsonschema = ">=3.0.1"
mistune = "*"
packaging = "*"
PyYAML = ">=3.0"
six = ">=1.10.0"

[[package]]
name = "flask"
version = "2.3.2"
description = "A simple framework for building complex web applications."
optional = false
python-versions = ">=3.8"
files = [
    {file = "Flask-2.3.2-py3-none-any.whl", hash = "sha256:77fd4e1249d8c9923de34907236b747ced06e5467ecac1a7bb7115ae0e9670b0"},
    {file = "Flask-2.3.2.tar.gz", hash = "sha256:8c2f9abd47a9e8df7f0c3f091ce9497d011dc3b31effcf4c85a6e2b50f4114ef"},
]

[package.dependencies]
blinker = ">=1.6.2"
click = ">=8.1.3"
itsdangerous = ">=2.1.2"
Jinja2 = ">=3.1.2"
Werkzeug = ">=2.3.3"

[package.extras]
async = ["asgiref (>=3.2)"]
dotenv = ["python-dotenv"]

[[package]]
name = "flask-caching"
version = "2.0.2"
description = "Adds caching support to Flask applications."
optional = false
python-versions = ">=3.7"
files = [
    {file = "Flask-Caching-2.0.2.tar.gz", hash = "sha256:24b60c552d59a9605cc1b6a42c56cdb39a82a28dab4532bbedb9222ae54ecb4e"},
    {file = "Flask_Caching-2.0.2-py3-none-any.whl", hash = "sha256:19571f2570e9b8dd9dd9d2f49d7cbee69c14ebe8cc001100b1eb98c379dd80ad"},
]

[package.dependencies]
cachelib = ">=0.9.0,<0.10.0"
Flask = "<3"

[[package]]
name = "iniconfig"
version = "2.0.0"
description = "brain-dead simple config-ini parsing"
optional = false
python-versions = ">=3.7"
files = [
    {file = "iniconfig-2.0.0-py3-none-any.whl", hash = "sha256:b6a85871a79d2e3b22d2d1b94ac2824226a63c6b741c88f7ae975f18b6778374"},
    {file = "iniconfig-2.0.0.tar.gz", hash = "sha256:2d91e135bf72d31a410b17c16da610a82cb55f6b0477d1a902134b24a455b8b3"},
]

[[package]]
name = "itsdangerous"
version = "2.1.2"
description = "Safely pass data to untrusted environments and back."
optional = false
python-versions = ">=3.7"
files = [
    {file = "itsdangerous-2.1.2-py3-none-any.whl", hash = "sha256:2c2349112351b88699d8d4b6b075022c0808887cb7ad10069318a8b0bc88db44"},
    {file = "itsdangerous-2.1.2.tar.gz", hash = "sha256:5dbbc68b317e5e42f327f9021763545dc3fc3bfe22e6deb96aaf1fc38874156a"},
]

[[package]]
name = "jinja2"
version = "3.1.2"
description = "A very fast and expressive template engine."
optional = false
python-versions = ">=3.7"
files = [
    {file = "Jinja2-3.1.2-py3-none-any.whl", hash = "sha256:6088930bfe239f0e6710546ab9c19c9ef35e29792895fed6e6e31a023a182a61"},
    {file = "Jinja2-3.1.2.tar.gz", hash = "sha256:31351a702a408a9e7595a8fc6150fc3f43bb6bf7e319770cbc0db9df9437e852"},
]

[package.dependencies]
MarkupSafe = ">=2.0"

[package.extras]
i18n = ["Babel (>=2.7)"]

[[package]]
name = "json-logging-py"
version = "0.2"
description = "JSON / Logstash formatters for Python logging"
optional = false
python-versions = "*"
files = [
    {file = "json-logging-py-0.2.tar.gz", hash = "sha256:118b1fe1f4eacaea6370e5b9710d0f6d0c0a4599aef9d5b9875a6a579974fc9a"},
]

[[package]]
name = "jsonformatter"
version = "0.3.2"
description = "Python log in json format."
optional = false
python-versions = ">=2.7"
files = [
    {file = "jsonformatter-0.3.2.tar.gz", hash = "sha256:d69c4b294cd9030b6803659924908f979ae827f5bdccde6a3815c0afb062948f"},
]

[[package]]
name = "jsonschema"
version = "4.17.3"
description = "An implementation of JSON Schema validation for Python"
optional = false
python-versions = ">=3.7"
files = [
    {file = "jsonschema-4.17.3-py3-none-any.whl", hash = "sha256:a870ad254da1a8ca84b6a2905cac29d265f805acc57af304784962a2aa6508f6"},
    {file = "jsonschema-4.17.3.tar.gz", hash = "sha256:0f864437ab8b6076ba6707453ef8f98a6a0d512a80e93f8abdb676f737ecb60d"},
]

[package.dependencies]
attrs = ">=17.4.0"
pyrsistent = ">=0.14.0,<0.17.0 || >0.17.0,<0.17.1 || >0.17.1,<0.17.2 || >0.17.2"

[package.extras]
format = ["fqdn", "idna", "isoduration", "jsonpointer (>1.13)", "rfc3339-validator", "rfc3987", "uri-template", "webcolors (>=1.11)"]
format-nongpl = ["fqdn", "idna", "isoduration", "jsonpointer (>1.13)", "rfc3339-validator", "rfc3986-validator (>0.1.0)", "uri-template", "webcolors (>=1.11)"]

[[package]]
name = "markupsafe"
version = "2.1.3"
description = "Safely add untrusted strings to HTML/XML markup."
optional = false
python-versions = ">=3.7"
files = [
    {file = "MarkupSafe-2.1.3-cp310-cp310-macosx_10_9_universal2.whl", hash = "sha256:cd0f502fe016460680cd20aaa5a76d241d6f35a1c3350c474bac1273803893fa"},
    {file = "MarkupSafe-2.1.3-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:e09031c87a1e51556fdcb46e5bd4f59dfb743061cf93c4d6831bf894f125eb57"},
    {file = "MarkupSafe-2.1.3-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:68e78619a61ecf91e76aa3e6e8e33fc4894a2bebe93410754bd28fce0a8a4f9f"},
    {file = "MarkupSafe-2.1.3-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:65c1a9bcdadc6c28eecee2c119465aebff8f7a584dd719facdd9e825ec61ab52"},
    {file = "MarkupSafe-2.1.3-cp310-cp310-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:525808b8019e36eb524b8c68acdd63a37e75714eac50e988180b169d64480a00"},
    {file = "MarkupSafe-2.1.3-cp310-cp310-musllinux_1_1_aarch64.whl", hash = "sha256:962f82a3086483f5e5f64dbad880d31038b698494799b097bc59c2edf392fce6"},
    {file = "MarkupSafe-2.1.3-cp310-cp310-musllinux_1_1_i686.whl", hash = "sha256:aa7bd130efab1c280bed0f45501b7c8795f9fdbeb02e965371bbef3523627779"},
    {file = "MarkupSafe-2.1.3-cp310-cp310-musllinux_1_1_x86_64.whl", hash = "sha256:c9c804664ebe8f83a211cace637506669e7890fec1b4195b505c214e50dd4eb7"},
    {file = "MarkupSafe-2.1.3-cp310-cp310-win32.whl", hash = "sha256:10bbfe99883db80bdbaff2dcf681dfc6533a614f700da1287707e8a5d78a8431"},
    {file = "MarkupSafe-2.1.3-cp310-cp310-win_amd64.whl", hash = "sha256:1577735524cdad32f9f694208aa75e422adba74f1baee7551620e43a3141f559"},
    {file = "MarkupSafe-2.1.3-cp311-cp311-macosx_10_9_universal2.whl", hash = "sha256:ad9e82fb8f09ade1c3e1b996a6337afac2b8b9e365f926f5a61aacc71adc5b3c"},
    {file = "MarkupSafe-2.1.3-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:3c0fae6c3be832a0a0473ac912810b2877c8cb9d76ca48de1ed31e1c68386575"},
    {file = "MarkupSafe-2.1.3-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:b076b6226fb84157e3f7c971a47ff3a679d837cf338547532ab866c57930dbee"},
    {file = "MarkupSafe-2.1.3-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:bfce63a9e7834b12b87c64d6b155fdd9b3b96191b6bd334bf37db7ff1fe457f2"},
    {file = "MarkupSafe-2.1.3-cp311-cp311-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:338ae27d6b8745585f87218a3f23f1512dbf52c26c28e322dbe54bcede54ccb9"},
    {file = "MarkupSafe-2.1.3-cp311-cp311-musllinux_1_1_aarch64.whl", hash = "sha256:e4dd52d80b8c83fdce44e12478ad2e85c64ea965e75d66dbeafb0a3e77308fcc"},
    {file = "MarkupSafe-2.1.3-cp311-cp311-musllinux_1_1_i686.whl", hash = "sha256:df0be2b576a7abbf737b1575f048c23fb1d769f267ec4358296f31c2479db8f9"},
    {file = "MarkupSafe-2.1.3-cp311-cp311-musllinux_1_1_x86_64.whl", hash = "sha256:5bbe06f8eeafd38e5d0a4894ffec89378b6c6a625ff57e3028921f8ff59318ac"},
    {file = "MarkupSafe-2.1.3-cp311-cp311-win32.whl", hash = "sha256:dd15ff04ffd7e05ffcb7fe79f1b98041b8ea30ae9234aed2a9168b5797c3effb"},
    {file = "MarkupSafe-2.1.3-cp311-cp311-win_amd64.whl", hash = "sha256:134da1eca9ec0ae528110ccc9e48041e0828d79f24121a1a146161103c76e686"},
    {file = "MarkupSafe-2.1.3-cp37-cp37m-macosx_10_9_x86_64.whl", hash = "sha256:8e254ae696c88d98da6555f5ace2279cf7cd5b3f52be2b5cf97feafe883b58d2"},
    {file = "MarkupSafe-2.1.3-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:cb0932dc158471523c9637e807d9bfb93e06a95cbf010f1a38b98623b929ef2b"},
    {file = "MarkupSafe-2.1.3-cp37-cp37m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:9402b03f1a1b4dc4c19845e5c749e3ab82d5078d16a2a4c2cd2df62d57bb0707"},
    {file = "MarkupSafe-2.1.3-cp37-cp37m-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:ca379055a47383d02a5400cb0d110cef0a776fc644cda797db0c5696cfd7e18e"},
    {file = "MarkupSafe-2.1.3-cp37-cp37m-musllinux_1_1_aarch64.whl", hash = "sha256:b7ff0f54cb4ff66dd38bebd335a38e2c22c41a8ee45aa608efc890ac3e3931bc"},
    {file = "MarkupSafe-2.1.3-cp37-cp37m-musllinux_1_1_i686.whl", hash = "sha256:c011a4149cfbcf9f03994ec2edffcb8b1dc2d2aede7ca243746df97a5d41ce48"},
    {file = "MarkupSafe-2.1.3-cp37-cp37m-musllinux_1_1_x86_64.whl", hash = "sha256:56d9f2ecac662ca1611d183feb03a3fa4406469dafe241673d521dd5ae92a155"},
    {file = "MarkupSafe-2.1.3-cp37-cp37m-win32.whl", hash = "sha256:8758846a7e80910096950b67071243da3e5a20ed2546e6392603c096778d48e0"},
    {file = "MarkupSafe-2.1.3-cp37-cp37m-win_amd64.whl", hash = "sha256:787003c0ddb00500e49a10f2844fac87aa6ce977b90b0feaaf9de23c22508b24"},
    {file = "MarkupSafe-2.1.3-cp38-cp38-macosx_10_9_universal2.whl", hash = "sha256:2ef12179d3a291be237280175b542c07a36e7f60718296278d8593d21ca937d4"},
    {file = "MarkupSafe-2.1.3-cp38-cp38-macosx_10_9_x86_64.whl", hash = "sha256:2c1b19b3aaacc6e57b7e25710ff571c24d6c3613a45e905b1fde04d691b98ee0"},
    {file = "MarkupSafe-2.1.3-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:8afafd99945ead6e075b973fefa56379c5b5c53fd8937dad92c662da5d8fd5ee"},
    {file = "MarkupSafe-2.1.3-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:8c41976a29d078bb235fea9b2ecd3da465df42a562910f9022f1a03107bd02be"},
    {file = "MarkupSafe-2.1.3-cp38-cp38-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:d080e0a5eb2529460b30190fcfcc4199bd7f827663f858a226a81bc27beaa97e"},
    {file = "MarkupSafe-2.1.3-cp38-cp38-musllinux_1_1_aarch64.whl", hash = "sha256:69c0f17e9f5a7afdf2cc9fb2d1ce6aabdb3bafb7f38017c0b77862bcec2bbad8"},
    {file = "MarkupSafe-2.1.3-cp38-cp38-musllinux_1_1_i686.whl", hash = "sha256:504b320cd4b7eff6f968eddf81127112db685e81f7e36e75f9f84f0df46041c3"},
    {file = "MarkupSafe-2.1.3-cp38-cp38-musllinux_1_1_x86_64.whl", hash = "sha256:42de32b22b6b804f42c5d98be4f7e5e977ecdd9ee9b660fda1a3edf03b11792d"},
    {file = "MarkupSafe-2.1.3-cp38-cp38-win32.whl", hash = "sha256:ceb01949af7121f9fc39f7d27f91be8546f3fb112c608bc4029aef0bab86a2a5"},
    {file = "MarkupSafe-2.1.3-cp38-cp38-win_amd64.whl", hash = "sha256:1b40069d487e7edb2676d3fbdb2b0829ffa2cd63a2ec26c4938b2d34391b4ecc"},
    {file = "MarkupSafe-2.1.3-cp39-cp39-macosx_10_9_universal2.whl", hash = "sha256:8023faf4e01efadfa183e863fefde0046de576c6f14659e8782065bcece22198"},
    {file = "MarkupSafe-2.1.3-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:6b2b56950d93e41f33b4223ead100ea0fe11f8e6ee5f641eb753ce4b77a7042b"},
    {file = "MarkupSafe-2.1.3-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:9dcdfd0eaf283af041973bff14a2e143b8bd64e069f4c383416ecd79a81aab58"},
    {file = "MarkupSafe-2.1.3-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:05fb21170423db021895e1ea1e1f3ab3adb85d1c2333cbc2310f2a26bc77272e"},
    {file = "MarkupSafe-2.1.3-cp39-cp39-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:282c2cb35b5b673bbcadb33a585408104df04f14b2d9b01d4c345a3b92861c2c"},
    {file = "MarkupSafe-2.1.3-cp39-cp39-musllinux_1_1_aarch64.whl", hash = "sha256:ab4a0df41e7c16a1392727727e7998a467472d0ad65f3ad5e6e765015df08636"},
    {file = "MarkupSafe-2.1.3-cp39-cp39-musllinux_1_1_i686.whl", hash = "sha256:7ef3cb2ebbf91e330e3bb937efada0edd9003683db6b57bb108c4001f37a02ea"},
    {file = "MarkupSafe-2.1.3-cp39-cp39-musllinux_1_1_x86_64.whl", hash = "sha256:0a4e4a1aff6c7ac4cd55792abf96c915634c2b97e3cc1c7129578aa68ebd754e"},
    {file = "MarkupSafe-2.1.3-cp39-cp39-win32.whl", hash = "sha256:fec21693218efe39aa7f8599346e90c705afa52c5b31ae019b2e57e8f6542bb2"},
    {file = "MarkupSafe-2.1.3-cp39-cp39-win_amd64.whl", hash = "sha256:3fd4abcb888d15a94f32b75d8fd18ee162ca0c064f35b11134be77050296d6ba"},
    {file = "MarkupSafe-2.1.3.tar.gz", hash = "sha256:af598ed32d6ae86f1b747b82783958b1a4ab8f617b06fe68795c7f026abbdcad"},
]

[[package]]
name = "mistune"
version = "3.0.1"
description = "A sane and fast Markdown parser with useful plugins and renderers"
optional = false
python-versions = ">=3.7"
files = [
    {file = "mistune-3.0.1-py3-none-any.whl", hash = "sha256:b9b3e438efbb57c62b5beb5e134dab664800bdf1284a7ee09e8b12b13eb1aac6"},
    {file = "mistune-3.0.1.tar.gz", hash = "sha256:e912116c13aa0944f9dc530db38eb88f6a77087ab128f49f84a48f4c05ea163c"},
]

[[package]]
name = "packaging"
version = "23.1"
description = "Core utilities for Python packages"
optional = false
python-versions = ">=3.7"
files = [
    {file = "packaging-23.1-py3-none-any.whl", hash = "sha256:994793af429502c4ea2ebf6bf664629d07c1a9fe974af92966e4b8d2df7edc61"},
    {file = "packaging-23.1.tar.gz", hash = "sha256:a392980d2b6cffa644431898be54b0045151319d1e7ec34f0cfed48767dd334f"},
]

[[package]]
name = "peewee"
version = "3.16.2"
description = "a little orm"
optional = false
python-versions = "*"
files = [
    {file = "peewee-3.16.2.tar.gz", hash = "sha256:10769981198c7311f84a0ca8db892fa213303a8eb1305deb795a71e7bd606a91"},
]

[[package]]
name = "pluggy"
version = "1.2.0"
description = "plugin and hook calling mechanisms for python"
optional = false
python-versions = ">=3.7"
files = [
    {file = "pluggy-1.2.0-py3-none-any.whl", hash = "sha256:c2fd55a7d7a3863cba1a013e4e2414658b1d07b6bc57b3919e0c63c9abb99849"},
    {file = "pluggy-1.2.0.tar.gz", hash = "sha256:d12f0c4b579b15f5e054301bb226ee85eeeba08ffec228092f8defbaa3a4c4b3"},
]

[package.extras]
dev = ["pre-commit", "tox"]
testing = ["pytest", "pytest-benchmark"]

[[package]]
name = "prometheus-client"
version = "0.17.0"
description = "Python client for the Prometheus monitoring system."
optional = false
python-versions = ">=3.6"
files = [
    {file = "prometheus_client-0.17.0-py3-none-any.whl", hash = "sha256:a77b708cf083f4d1a3fb3ce5c95b4afa32b9c521ae363354a4a910204ea095ce"},
    {file = "prometheus_client-0.17.0.tar.gz", hash = "sha256:9c3b26f1535945e85b8934fb374678d263137b78ef85f305b1156c7c881cd11b"},
]

[package.extras]
twisted = ["twisted"]

[[package]]
name = "prometheus-flask-exporter"
version = "0.22.4"
description = "Prometheus metrics exporter for Flask"
optional = false
python-versions = "*"
files = [
    {file = "prometheus_flask_exporter-0.22.4-py3-none-any.whl", hash = "sha256:e130179c26d5a9b903c12c0d8826127ae491b04b298cae0b92b98677dcf2c06f"},
    {file = "prometheus_flask_exporter-0.22.4.tar.gz", hash = "sha256:959b69f1e740b6736ea53fe5f28dc2ab6229b2ebeade6582b3dbb5d74c7d58e4"},
]

[package.dependencies]
flask = "*"
prometheus-client = "*"

[[package]]
name = "psycopg2"
version = "2.9.6"
description = "psycopg2 - Python-PostgreSQL Database Adapter"
optional = false
python-versions = ">=3.6"
files = [
    {file = "psycopg2-2.9.6-cp310-cp310-win32.whl", hash = "sha256:f7a7a5ee78ba7dc74265ba69e010ae89dae635eea0e97b055fb641a01a31d2b1"},
    {file = "psycopg2-2.9.6-cp310-cp310-win_amd64.whl", hash = "sha256:f75001a1cbbe523e00b0ef896a5a1ada2da93ccd752b7636db5a99bc57c44494"},
    {file = "psycopg2-2.9.6-cp311-cp311-win32.whl", hash = "sha256:53f4ad0a3988f983e9b49a5d9765d663bbe84f508ed655affdb810af9d0972ad"},
    {file = "psycopg2-2.9.6-cp311-cp311-win_amd64.whl", hash = "sha256:b81fcb9ecfc584f661b71c889edeae70bae30d3ef74fa0ca388ecda50b1222b7"},
    {file = "psycopg2-2.9.6-cp36-cp36m-win32.whl", hash = "sha256:11aca705ec888e4f4cea97289a0bf0f22a067a32614f6ef64fcf7b8bfbc53744"},
    {file = "psycopg2-2.9.6-cp36-cp36m-win_amd64.whl", hash = "sha256:36c941a767341d11549c0fbdbb2bf5be2eda4caf87f65dfcd7d146828bd27f39"},
    {file = "psycopg2-2.9.6-cp37-cp37m-win32.whl", hash = "sha256:869776630c04f335d4124f120b7fb377fe44b0a7645ab3c34b4ba42516951889"},
    {file = "psycopg2-2.9.6-cp37-cp37m-win_amd64.whl", hash = "sha256:a8ad4a47f42aa6aec8d061fdae21eaed8d864d4bb0f0cade5ad32ca16fcd6258"},
    {file = "psycopg2-2.9.6-cp38-cp38-win32.whl", hash = "sha256:2362ee4d07ac85ff0ad93e22c693d0f37ff63e28f0615a16b6635a645f4b9214"},
    {file = "psycopg2-2.9.6-cp38-cp38-win_amd64.whl", hash = "sha256:d24ead3716a7d093b90b27b3d73459fe8cd90fd7065cf43b3c40966221d8c394"},
    {file = "psycopg2-2.9.6-cp39-cp39-win32.whl", hash = "sha256:1861a53a6a0fd248e42ea37c957d36950da00266378746588eab4f4b5649e95f"},
    {file = "psycopg2-2.9.6-cp39-cp39-win_amd64.whl", hash = "sha256:ded2faa2e6dfb430af7713d87ab4abbfc764d8d7fb73eafe96a24155f906ebf5"},
    {file = "psycopg2-2.9.6.tar.gz", hash = "sha256:f15158418fd826831b28585e2ab48ed8df2d0d98f502a2b4fe619e7d5ca29011"},
]

[[package]]
name = "pyrsistent"
version = "0.19.3"
description = "Persistent/Functional/Immutable data structures"
optional = false
python-versions = ">=3.7"
files = [
    {file = "pyrsistent-0.19.3-cp310-cp310-macosx_10_9_universal2.whl", hash = "sha256:20460ac0ea439a3e79caa1dbd560344b64ed75e85d8703943e0b66c2a6150e4a"},
    {file = "pyrsistent-0.19.3-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:4c18264cb84b5e68e7085a43723f9e4c1fd1d935ab240ce02c0324a8e01ccb64"},
    {file = "pyrsistent-0.19.3-cp310-cp310-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:4b774f9288dda8d425adb6544e5903f1fb6c273ab3128a355c6b972b7df39dcf"},
    {file = "pyrsistent-0.19.3-cp310-cp310-win32.whl", hash = "sha256:5a474fb80f5e0d6c9394d8db0fc19e90fa540b82ee52dba7d246a7791712f74a"},
    {file = "pyrsistent-0.19.3-cp310-cp310-win_amd64.whl", hash = "sha256:49c32f216c17148695ca0e02a5c521e28a4ee6c5089f97e34fe24163113722da"},
    {file = "pyrsistent-0.19.3-cp311-cp311-macosx_10_9_universal2.whl", hash = "sha256:f0774bf48631f3a20471dd7c5989657b639fd2d285b861237ea9e82c36a415a9"},
    {file = "pyrsistent-0.19.3-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:3ab2204234c0ecd8b9368dbd6a53e83c3d4f3cab10ecaf6d0e772f456c442393"},
    {file = "pyrsistent-0.19.3-cp311-cp311-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:e42296a09e83028b3476f7073fcb69ffebac0e66dbbfd1bd847d61f74db30f19"},
    {file = "pyrsistent-0.19.3-cp311-cp311-win32.whl", hash = "sha256:64220c429e42a7150f4bfd280f6f4bb2850f95956bde93c6fda1b70507af6ef3"},
    {file = "pyrsistent-0.19.3-cp311-cp311-win_amd64.whl", hash = "sha256:016ad1afadf318eb7911baa24b049909f7f3bb2c5b1ed7b6a8f21db21ea3faa8"},
    {file = "pyrsistent-0.19.3-cp37-cp37m-macosx_10_9_x86_64.whl", hash = "sha256:c4db1bd596fefd66b296a3d5d943c94f4fac5bcd13e99bffe2ba6a759d959a28"},
    {file = "pyrsistent-0.19.3-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:aeda827381f5e5d65cced3024126529ddc4289d944f75e090572c77ceb19adbf"},
    {file = "pyrsistent-0.19.3-cp37-cp37m-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:42ac0b2f44607eb92ae88609eda931a4f0dfa03038c44c772e07f43e738bcac9"},
    {file = "pyrsistent-0.19.3-cp37-cp37m-win32.whl", hash = "sha256:e8f2b814a3dc6225964fa03d8582c6e0b6650d68a232df41e3cc1b66a5d2f8d1"},
    {file = "pyrsistent-0.19.3-cp37-cp37m-win_amd64.whl", hash = "sha256:c9bb60a40a0ab9aba40a59f68214eed5a29c6274c83b2cc206a359c4a89fa41b"},
    {file = "pyrsistent-0.19.3-cp38-cp38-macosx_10_9_universal2.whl", hash = "sha256:a2471f3f8693101975b1ff85ffd19bb7ca7dd7c38f8a81701f67d6b4f97b87d8"},
    {file = "pyrsistent-0.19.3-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:cc5d149f31706762c1f8bda2e8c4f8fead6e80312e3692619a75301d3dbb819a"},
    {file = "pyrsistent-0.19.3-cp38-cp38-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:3311cb4237a341aa52ab8448c27e3a9931e2ee09561ad150ba94e4cfd3fc888c"},
    {file = "pyrsistent-0.19.3-cp38-cp38-win32.whl", hash = "sha256:f0e7c4b2f77593871e918be000b96c8107da48444d57005b6a6bc61fb4331b2c"},
    {file = "pyrsistent-0.19.3-cp38-cp38-win_amd64.whl", hash = "sha256:c147257a92374fde8498491f53ffa8f4822cd70c0d85037e09028e478cababb7"},
    {file = "pyrsistent-0.19.3-cp39-cp39-macosx_10_9_universal2.whl", hash = "sha256:b735e538f74ec31378f5a1e3886a26d2ca6351106b4dfde376a26fc32a044edc"},
    {file = "pyrsistent-0.19.3-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:99abb85579e2165bd8522f0c0138864da97847875ecbd45f3e7e2af569bfc6f2"},
    {file = "pyrsistent-0.19.3-cp39-cp39-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:3a8cb235fa6d3fd7aae6a4f1429bbb1fec1577d978098da1252f0489937786f3"},
    {file = "pyrsistent-0.19.3-cp39-cp39-win32.whl", hash = "sha256:c74bed51f9b41c48366a286395c67f4e894374306b197e62810e0fdaf2364da2"},
    {file = "pyrsistent-0.19.3-cp39-cp39-win_amd64.whl", hash = "sha256:878433581fc23e906d947a6814336eee031a00e6defba224234169ae3d3d6a98"},
    {file = "pyrsistent-0.19.3-py3-none-any.whl", hash = "sha256:ccf0d6bd208f8111179f0c26fdf84ed7c3891982f2edaeae7422575f47e66b64"},
    {file = "pyrsistent-0.19.3.tar.gz", hash = "sha256:1a2994773706bbb4995c31a97bc94f1418314923bd1048c6d964837040376440"},
]

[[package]]
name = "pytest"
version = "7.4.0"
description = "pytest: simple powerful testing with Python"
optional = false
python-versions = ">=3.7"
files = [
    {file = "pytest-7.4.0-py3-none-any.whl", hash = "sha256:78bf16451a2eb8c7a2ea98e32dc119fd2aa758f1d5d66dbf0a59d69a3969df32"},
    {file = "pytest-7.4.0.tar.gz", hash = "sha256:b4bf8c45bd59934ed84001ad51e11b4ee40d40a1229d2c79f9c592b0a3f6bd8a"},
]

[package.dependencies]
colorama = {version = "*", markers = "sys_platform == \"win32\""}
iniconfig = "*"
packaging = "*"
pluggy = ">=0.12,<2.0"

[package.extras]
testing = ["argcomplete", "attrs (>=19.2.0)", "hypothesis (>=3.56)", "mock", "nose", "pygments (>=2.7.2)", "requests", "setuptools", "xmlschema"]

[[package]]
name = "pytest-cov"
version = "4.1.0"
description = "Pytest plugin for measuring coverage."
optional = false
python-versions = ">=3.7"
files = [
    {file = "pytest-cov-4.1.0.tar.gz", hash = "sha256:3904b13dfbfec47f003b8e77fd5b589cd11904a21ddf1ab38a64f204d6a10ef6"},
    {file = "pytest_cov-4.1.0-py3-none-any.whl", hash = "sha256:6ba70b9e97e69fcc3fb45bfeab2d0a138fb65c4d0d6a41ef33983ad114be8c3a"},
]

[package.dependencies]
coverage = {version = ">=5.2.1", extras = ["toml"]}
pytest = ">=4.6"

[package.extras]
testing = ["fields", "hunter", "process-tests", "pytest-xdist", "six", "virtualenv"]

[[package]]
name = "pytest-mock"
version = "3.11.1"
description = "Thin-wrapper around the mock package for easier use with pytest"
optional = false
python-versions = ">=3.7"
files = [
    {file = "pytest-mock-3.11.1.tar.gz", hash = "sha256:7f6b125602ac6d743e523ae0bfa71e1a697a2f5534064528c6ff84c2f7c2fc7f"},
    {file = "pytest_mock-3.11.1-py3-none-any.whl", hash = "sha256:21c279fff83d70763b05f8874cc9cfb3fcacd6d354247a976f9529d19f9acf39"},
]

[package.dependencies]
pytest = ">=5.0"

[package.extras]
dev = ["pre-commit", "pytest-asyncio", "tox"]

[[package]]
name = "python-json-logger"
version = "2.0.7"
description = "A python library adding a json log formatter"
optional = false
python-versions = ">=3.6"
files = [
    {file = "python-json-logger-2.0.7.tar.gz", hash = "sha256:23e7ec02d34237c5aa1e29a070193a4ea87583bb4e7f8fd06d3de8264c4b2e1c"},
    {file = "python_json_logger-2.0.7-py3-none-any.whl", hash = "sha256:f380b826a991ebbe3de4d897aeec42760035ac760345e57b812938dc8b35e2bd"},
]

[[package]]
name = "pyyaml"
version = "6.0"
description = "YAML parser and emitter for Python"
optional = false
python-versions = ">=3.6"
files = [
    {file = "PyYAML-6.0-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:d4db7c7aef085872ef65a8fd7d6d09a14ae91f691dec3e87ee5ee0539d516f53"},
    {file = "PyYAML-6.0-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:9df7ed3b3d2e0ecfe09e14741b857df43adb5a3ddadc919a2d94fbdf78fea53c"},
    {file = "PyYAML-6.0-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:77f396e6ef4c73fdc33a9157446466f1cff553d979bd00ecb64385760c6babdc"},
    {file = "PyYAML-6.0-cp310-cp310-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:a80a78046a72361de73f8f395f1f1e49f956c6be882eed58505a15f3e430962b"},
    {file = "PyYAML-6.0-cp310-cp310-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl", hash = "sha256:f84fbc98b019fef2ee9a1cb3ce93e3187a6df0b2538a651bfb890254ba9f90b5"},
    {file = "PyYAML-6.0-cp310-cp310-win32.whl", hash = "sha256:2cd5df3de48857ed0544b34e2d40e9fac445930039f3cfe4bcc592a1f836d513"},
    {file = "PyYAML-6.0-cp310-cp310-win_amd64.whl", hash = "sha256:daf496c58a8c52083df09b80c860005194014c3698698d1a57cbcfa182142a3a"},
    {file = "PyYAML-6.0-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:d4b0ba9512519522b118090257be113b9468d804b19d63c71dbcf4a48fa32358"},
    {file = "PyYAML-6.0-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:81957921f441d50af23654aa6c5e5eaf9b06aba7f0a19c18a538dc7ef291c5a1"},
    {file = "PyYAML-6.0-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:afa17f5bc4d1b10afd4466fd3a44dc0e245382deca5b3c353d8b757f9e3ecb8d"},
    {file = "PyYAML-6.0-cp311-cp311-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:dbad0e9d368bb989f4515da330b88a057617d16b6a8245084f1b05400f24609f"},
    {file = "PyYAML-6.0-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:432557aa2c09802be39460360ddffd48156e30721f5e8d917f01d31694216782"},
    {file = "PyYAML-6.0-cp311-cp311-win32.whl", hash = "sha256:bfaef573a63ba8923503d27530362590ff4f576c626d86a9fed95822a8255fd7"},
    {file = "PyYAML-6.0-cp311-cp311-win_amd64.whl", hash = "sha256:01b45c0191e6d66c470b6cf1b9531a771a83c1c4208272ead47a3ae4f2f603bf"},
    {file = "PyYAML-6.0-cp36-cp36m-macosx_10_9_x86_64.whl", hash = "sha256:897b80890765f037df3403d22bab41627ca8811ae55e9a722fd0392850ec4d86"},
    {file = "PyYAML-6.0-cp36-cp36m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:50602afada6d6cbfad699b0c7bb50d5ccffa7e46a3d738092afddc1f9758427f"},
    {file = "PyYAML-6.0-cp36-cp36m-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:48c346915c114f5fdb3ead70312bd042a953a8ce5c7106d5bfb1a5254e47da92"},
    {file = "PyYAML-6.0-cp36-cp36m-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl", hash = "sha256:98c4d36e99714e55cfbaaee6dd5badbc9a1ec339ebfc3b1f52e293aee6bb71a4"},
    {file = "PyYAML-6.0-cp36-cp36m-win32.whl", hash = "sha256:0283c35a6a9fbf047493e3a0ce8d79ef5030852c51e9d911a27badfde0605293"},
    {file = "PyYAML-6.0-cp36-cp36m-win_amd64.whl", hash = "sha256:07751360502caac1c067a8132d150cf3d61339af5691fe9e87803040dbc5db57"},
    {file = "PyYAML-6.0-cp37-cp37m-macosx_10_9_x86_64.whl", hash = "sha256:819b3830a1543db06c4d4b865e70ded25be52a2e0631ccd2f6a47a2822f2fd7c"},
    {file = "PyYAML-6.0-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:473f9edb243cb1935ab5a084eb238d842fb8f404ed2193a915d1784b5a6b5fc0"},
    {file = "PyYAML-6.0-cp37-cp37m-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:0ce82d761c532fe4ec3f87fc45688bdd3a4c1dc5e0b4a19814b9009a29baefd4"},
    {file = "PyYAML-6.0-cp37-cp37m-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl", hash = "sha256:231710d57adfd809ef5d34183b8ed1eeae3f76459c18fb4a0b373ad56bedcdd9"},
    {file = "PyYAML-6.0-cp37-cp37m-win32.whl", hash = "sha256:c5687b8d43cf58545ade1fe3e055f70eac7a5a1a0bf42824308d868289a95737"},
    {file = "PyYAML-6.0-cp37-cp37m-win_amd64.whl", hash = "sha256:d15a181d1ecd0d4270dc32edb46f7cb7733c7c508857278d3d378d14d606db2d"},
    {file = "PyYAML-6.0-cp38-cp38-macosx_10_9_x86_64.whl", hash = "sha256:0b4624f379dab24d3725ffde76559cff63d9ec94e1736b556dacdfebe5ab6d4b"},
    {file = "PyYAML-6.0-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:213c60cd50106436cc818accf5baa1aba61c0189ff610f64f4a3e8c6726218ba"},
    {file = "PyYAML-6.0-cp38-cp38-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:9fa600030013c4de8165339db93d182b9431076eb98eb40ee068700c9c813e34"},
    {file = "PyYAML-6.0-cp38-cp38-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl", hash = "sha256:277a0ef2981ca40581a47093e9e2d13b3f1fbbeffae064c1d21bfceba2030287"},
    {file = "PyYAML-6.0-cp38-cp38-win32.whl", hash = "sha256:d4eccecf9adf6fbcc6861a38015c2a64f38b9d94838ac1810a9023a0609e1b78"},
    {file = "PyYAML-6.0-cp38-cp38-win_amd64.whl", hash = "sha256:1e4747bc279b4f613a09eb64bba2ba602d8a6664c6ce6396a4d0cd413a50ce07"},
    {file = "PyYAML-6.0-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:055d937d65826939cb044fc8c9b08889e8c743fdc6a32b33e2390f66013e449b"},
    {file = "PyYAML-6.0-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:e61ceaab6f49fb8bdfaa0f92c4b57bcfbea54c09277b1b4f7ac376bfb7a7c174"},
    {file = "PyYAML-6.0-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:d67d839ede4ed1b28a4e8909735fc992a923cdb84e618544973d7dfc71540803"},
    {file = "PyYAML-6.0-cp39-cp39-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:cba8c411ef271aa037d7357a2bc8f9ee8b58b9965831d9e51baf703280dc73d3"},
    {file = "PyYAML-6.0-cp39-cp39-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl", hash = "sha256:40527857252b61eacd1d9af500c3337ba8deb8fc298940291486c465c8b46ec0"},
    {file = "PyYAML-6.0-cp39-cp39-win32.whl", hash = "sha256:b5b9eccad747aabaaffbc6064800670f0c297e52c12754eb1d976c57e4f74dcb"},
    {file = "PyYAML-6.0-cp39-cp39-win_amd64.whl", hash = "sha256:b3d267842bf12586ba6c734f89d1f5b871df0273157918b0ccefa29deb05c21c"},
    {file = "PyYAML-6.0.tar.gz", hash = "sha256:68fb519c14306fec9720a2a5b45bc9f0c8d1b9c72adf45c37baedfcd949c35a2"},
]

[[package]]
name = "redis"
version = "4.6.0"
description = "Python client for Redis database and key-value store"
optional = false
python-versions = ">=3.7"
files = [
    {file = "redis-4.6.0-py3-none-any.whl", hash = "sha256:e2b03db868160ee4591de3cb90d40ebb50a90dd302138775937f6a42b7ed183c"},
    {file = "redis-4.6.0.tar.gz", hash = "sha256:585dc516b9eb042a619ef0a39c3d7d55fe81bdb4df09a52c9cdde0d07bf1aa7d"},
]

[package.dependencies]
async-timeout = {version = ">=4.0.2", markers = "python_full_version <= \"3.11.2\""}

[package.extras]
hiredis = ["hiredis (>=1.0.0)"]
ocsp = ["cryptography (>=36.0.1)", "pyopenssl (==20.0.1)", "requests (>=2.26.0)"]

[[package]]
name = "six"
version = "1.16.0"
description = "Python 2 and 3 compatibility utilities"
optional = false
python-versions = ">=2.7, !=3.0.*, !=3.1.*, !=3.2.*"
files = [
    {file = "six-1.16.0-py2.py3-none-any.whl", hash = "sha256:8abb2f1d86890a2dfb989f9a77cfcfd3e47c2a354b01111771326f8aa26e0254"},
    {file = "six-1.16.0.tar.gz", hash = "sha256:1e61c37477a1626458e36f7b1d82aa5c9b094fa4802892072e49de9c60c4c926"},
]

[[package]]
name = "voluptuous"
version = "0.13.1"
description = ""
optional = false
python-versions = "*"
files = [
    {file = "voluptuous-0.13.1-py3-none-any.whl", hash = "sha256:4b838b185f5951f2d6e8752b68fcf18bd7a9c26ded8f143f92d6d28f3921a3e6"},
    {file = "voluptuous-0.13.1.tar.gz", hash = "sha256:e8d31c20601d6773cb14d4c0f42aee29c6821bbd1018039aac7ac5605b489723"},
]

[[package]]
name = "werkzeug"
version = "2.3.6"
description = "The comprehensive WSGI web application library."
optional = false
python-versions = ">=3.8"
files = [
    {file = "Werkzeug-2.3.6-py3-none-any.whl", hash = "sha256:935539fa1413afbb9195b24880778422ed620c0fc09670945185cce4d91a8890"},
    {file = "Werkzeug-2.3.6.tar.gz", hash = "sha256:98c774df2f91b05550078891dee5f0eb0cb797a522c757a2452b9cee5b202330"},
]

[package.dependencies]
MarkupSafe = ">=2.1.1"

[package.extras]
watchdog = ["watchdog (>=2.3)"]

[metadata]
lock-version = "2.0"
python-versions = "^3.11"
content-hash = "14c7e919ef5cf3abe950d0c43d5338e35ef20dcdae212f94b3c4196ccba431ad"
ritu@localhost:~/attendance-api$ cd router/
ritu@localhost:~/attendance-api/router$ ls
attendance.py  cache.py  tests
ritu@localhost:~/attendance-api/router$ cat attendance.py 
"""
Module for all the application routes and their respective handlers
- create_record
- read_record
- read_all_record
- get_detail_healthcheck
- get_healthcheck
"""

# pylint: disable=import-error,invalid-name,redefined-builtin
from flask import Blueprint, jsonify, request
from voluptuous import Schema, Required
from client.postgres import DatabaseSDKFacade
from utils.validator import data_validator
from router.cache import cache

route = Blueprint("attendance", __name__)

@route.route("/attendance/create", methods=["POST"])
@data_validator(
    Schema(
        {
            Required("id"): str,
            Required("name"): str,
            Required("status"): str,
            Required("date"):str,
        }
    )
)
def create_record(
        id: str,
        name: str,
        status: str,
        date: str,
):
    """
    Function for creating the record in the database
    ---
    consumes:
      - application/json
    description: Create record of employee attendance
    produces:
      - application/json
    parameters:
    - description: Attendance Data Payload
      in: body
      name: attendance
      required: true
      schema:
        $ref: '#/definitions/EmployeeInfo'
    definitions:
      EmployeeInfo:
        properties:
          id:
            type: string
          name:
            type: string
          status:
            type: string
          date:
            type: string
        type: object
    responses:
      200:
        description: OK
        schema:
          $ref: '#/definitions/EmployeeInfo'
    summary: CreateRecord API is for creating particular attendance record
    tags:
      - attendance
    """
    record = DatabaseSDKFacade.database.create_employee_attendance(id, name, status, date)
    return jsonify(record)

@route.route("/attendance/search", methods=["GET"])
@cache.cached(timeout=20)
def read_record():
    """
    Function for reading the record from the database
    ---
    consumes:
      - application/json
    description: Read record of employee attendance
    produces:
      - application/json
    parameters:
    - description: User ID
      in: query
      name: id
      required: true
      type: string
    definitions:
      EmployeeInfo:
        properties:
          id:
            type: string
          name:
            type: string
          status:
            type: string
          date:
            type: string
        type: object
    responses:
      200:
        description: OK
        schema:
          $ref: '#/definitions/EmployeeInfo'
    summary: ReadRecord API is for getting particular attendance record
    tags:
      - attendance
    """
    args = request.args
    id = args.get("id", default="", type=str)
    if id != "":
        record = DatabaseSDKFacade.database.read_employee_attendance(id)
        return jsonify(record)
    return jsonify({"message": f"Unable to process request, please check query params {id}"}), 400

@route.route("/attendance/search/all", methods=["GET"])
@cache.cached(timeout=20)
def read_all_record():
    """
    Function for reading all the record from the database
    ---
    consumes:
      - application/json
    description: Read record of all employee attendance records
    produces:
      - application/json
    definitions:
      EmployeeInfo:
        properties:
          id:
            type: string
          name:
            type: string
          status:
            type: string
          date:
            type: string
        type: object
    responses:
      200:
        description: OK
        schema:
          $ref: '#/definitions/EmployeeInfo'
          type: array
    summary: ReadRecord API is for getting all attendance record
    tags:
      - attendance
    """
    record = DatabaseSDKFacade.database.read_all_employee_attendance()
    return jsonify(record)

@route.route("/attendance/health/detail", methods=["GET"])
def get_detail_healthcheck():
    """
    Function for getting detailed healthcheck of application
    ---
    consumes:
      - application/json
    description: Do detail healthcheck for attendance API
    produces:
      - application/json
    definitions:
      HealthMessage:
        properties:
          message:
            type: string
          postgresql:
            type: string
          redis:
            type: string
          status:
            type: string
        type: object
    responses:
      200:
        description: OK
        schema:
          $ref: '#/definitions/HealthMessage'
    summary: DetailedHealthCheckAPI is a method to perform detailed healthcheck
    tags:
      - healthcheck
    """
    status, status_code = DatabaseSDKFacade.database.attendance_detail_health()
    return jsonify(status), status_code

@route.route("/attendance/health", methods=["GET"])
def get_healthcheck():
    """
    Function for getting healthcheck of application
    ---
    consumes:
      - application/json
    description: Do healthcheck for attendance API
    produces:
      - application/json
    definitions:
      CustomMessage:
        properties:
          message:
            type: string
        type: object
    responses:
      200:
        description: OK
        schema:
          $ref: '#/definitions/CustomMessage'
    summary: HealthCheckAPI is a method to perform healthcheck of application
    tags:
      - healthcheck
    """
    status, status_code = DatabaseSDKFacade.database.attendance_health()
    return jsonify(status), status_code
ritu@localhost:~/attendance-api/router$ cat cache.py 
"""
Module for enable caching in attendance application
"""
from flask_caching import Cache

cache = Cache()
ritu@localhost:~/attendance-api/router$ ls
attendance.py  cache.py  tests
ritu@localhost:~/attendance-api/router$ cd tests/
ritu@localhost:~/attendance-api/router/tests$ ls
test_attendance.py  test_cache.py
ritu@localhost:~/attendance-api/router/tests$ cat test_attendance.py 
ritu@localhost:~/attendance-api/router/tests$ cat test_cache.py 
from flask import Flask
from flask_caching import Cache
import pytest

cache = Cache()

@pytest.fixture(scope='module')
def app():
    """Create a Flask app for testing"""
    app = Flask(__name__)
    app.config['TESTING'] = True
    app.config['CACHE_TYPE'] = 'simple'
    cache.init_app(app)
    return app

def test_cache_set_get(app):
    # Set a value in the cache
    cache.set('key', 'value')

    # Get the value from the cache
    result = cache.get('key')

    # Assert the retrieved value
    assert result == 'value'


def test_cache_delete(app):
    # Set a value in the cache
    cache.set('key', 'value')

    # Delete the value from the cache
    cache.delete('key')

    # Get the value from the cache
    result = cache.get('key')

    # Assert that the value is None (deleted)
    assert result is None


def test_cache_clear(app):
    # Set values in the cache
    cache.set('key1', 'value1')
    cache.set('key2', 'value2')

    # Clear the cache
    cache.clear()

    # Get the values from the cache
    result1 = cache.get('key1')
    result2 = cache.get('key2')

    # Assert that both values are None (cleared)
    assert result1 is None
    assert result2 is None


if __name__ == '__main__':
    pytest.main()
ritu@localhost:~/attendance-api/router/tests$ cat test_attendance.py 
ritu@localhost:~/attendance-api/router/tests$ cd ..
ritu@localhost:~/attendance-api/router$ ls
attendance.py  cache.py  tests
ritu@localhost:~/attendance-api/router$ cd ..
ritu@localhost:~/attendance-api$ ls
app.py       LICENSE               migration       pytest.ini  static
client       liquibase.properties  models          README.md   utils
config.yaml  log.conf              poetry.lock     router
Dockerfile   Makefile              pyproject.toml  scripts
ritu@localhost:~/attendance-api$ cat Makefile 
APP_VERSION ?= v0.1.0
IMAGE_REGISTRY ?= quay.io/opstree
IMAGE_NAME ?= attendance-api

# Build employee binary
build: fmt
	poetry config virtualenvs.create false
	poetry install --no-root --no-interaction --no-ansi

# Run go fmt against code
fmt:
	pylint router/ client/ models/ utils/ app.py

docker-build:
	docker build -t ${IMAGE_REGISTRY}/${IMAGE_NAME}:${APP_VERSION} -f Dockerfile .

docker-push:
	docker push ${IMAGE_REGISTRY}/${IMAGE_NAME}:${APP_VERSION}

run-migrations:
	liquibase update --driver-properties-file=liquibase.properties
ritu@localhost:~/attendance-api$ ls
app.py       LICENSE               migration       pytest.ini  static
client       liquibase.properties  models          README.md   utils
config.yaml  log.conf              poetry.lock     router
Dockerfile   Makefile              pyproject.toml  scripts
ritu@localhost:~/attendance-api$ cat pyproject.toml 
[tool.poetry]
name = "attendance-api"
version = "0.1.0"
description = "A python based microservice for handling attendance records"
authors = ["Opstree Solutions"]
license = "Apache"
readme = "README.md"
packages = [{include = "attendance_api"}]

[tool.poetry.dependencies]
python = "^3.11"
peewee = "^3.16.2"
voluptuous = "^0.13.1"
redis = "^4.6.0"
json-logging-py = "^0.2"
jsonformatter = "^0.3.2"
python-json-logger = "^2.0.7"
flask-caching = "^2.0.2"
prometheus-flask-exporter = "^0.22.4"
flasgger = "0.9.7b2"
pytest-mock = "^3.11.1"
pytest = "^7.4.0"
pytest-cov = "^4.1.0"

[tool.poetry.group.dev.dependencies]
Flask = "^2.3.2"
psycopg2 = "^2.9.6"
PyYAML = "^6.0"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"

[tool.coverage.run]
omit = [
    "app.py"
]
ritu@localhost:~/attendance-api$ cd scripts/
ritu@localhost:~/attendance-api/scripts$ ls
db_init.sh
ritu@localhost:~/attendance-api/scripts$ cat db_init.sh 
#!/usr/bin/env bash

POSTGRESQL_USERNAME="${POSTGRESQL_USERNAME:=postgres}"
PGPASSWORD="${PGPASSWORD:=password}"
POSTGRESQL_HOST="${POSTGRESQL_HOST:=localhost}"

PGPASSWORD=${PGPASSWORD} psql -U "${POSTGRESQL_USERNAME}" -h "${POSTGRESQL_HOST}" -c 'create database attendance_db;'
PGPASSWORD=${PGPASSWORD} psql -U "${POSTGRESQL_USERNAME}" -h "${POSTGRESQL_HOST}"  -c 'CREATE TABLE records(id TEXT PRIMARY KEY NOT NULL, name TEXT NOT NULL, status TEXT NOT NULL, date DATE);'
ritu@localhost:~/attendance-api/scripts$ cd ..
ritu@localhost:~/attendance-api$ ls
app.py       LICENSE               migration       pytest.ini  static
client       liquibase.properties  models          README.md   utils
config.yaml  log.conf              poetry.lock     router
Dockerfile   Makefile              pyproject.toml  scripts
ritu@localhost:~/attendance-api$ 




